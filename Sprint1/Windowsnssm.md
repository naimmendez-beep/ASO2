---
layout: default
title: "SISTEMES D'INICI"
---
# SISTEMES D'INICI A WINDOWS

- Aturar servei

<img width="826" height="509" alt="imatge" src="https://github.com/user-attachments/assets/bb35eeea-de28-4054-9d1f-76ec1f3d8cbf" />

- NSSM

<img width="409" height="124" alt="image" src="https://github.com/user-attachments/assets/17e4f17a-b8e2-4b41-8bcb-1d7f72e7ce27" />

##  TASCA AVALUABLE: Windows Sentinel 

<img width="593" height="204" alt="imatge" src="https://github.com/user-attachments/assets/25135cb1-c458-4cb8-b492-39a633b3e9a9" />

<img width="568" height="293" alt="imatge" src="https://github.com/user-attachments/assets/19c6bf95-02aa-409a-9c69-a5ff0874b646" />

<img width="402" height="225" alt="imatge" src="https://github.com/user-attachments/assets/95003fc5-7420-4f34-8c2f-b331d1fa15da" />

<img width="400" height="35" alt="imatge" src="https://github.com/user-attachments/assets/ea81f2bd-710c-4b36-af11-57234d0269e9" />

- "Contingut de l'arxiu creat"

param ([int]$Port = 8080)

$listener = New-Object System.Net.HttpListener
$listener.Prefixes.Add("http://localhost:$Port/")

try {
    $listener.Start()
    Write-Host "[+] Sentinel Server actiu a http://localhost:$Port/" -ForegroundColor Green
} catch {
    Write-Error "[-] Error en iniciar l'HttpListener: $_"
    exit 1
}

try {
    while ($listener.IsListening) {
        $context = $listener.GetContext()
        $request = $context.Request
        $response = $context.Response
        $url = $request.Url.AbsolutePath

        if ($url -eq "/" -or $url -eq "/index.html") {
            $htmlPath = "C:\Sentinel\index.html"
            if (Test-Path $htmlPath) {
                $content = [System.IO.File]::ReadAllBytes($htmlPath)
                $response.ContentType = "text/html; charset=utf-8"
                $response.ContentLength64 = $content.Length
                $response.OutputStream.Write($content, 0, $content.Length)
            } else {
                $notFound = [System.Text.Encoding]::UTF8.GetBytes("<h1>404 - index.html no trobat</h1>")
                $response.StatusCode = 404
                $response.OutputStream.Write($notFound, 0, $notFound.Length)
            }
        } 
        elseif ($url -eq "/api/metrics") {
            $topProcesses = Get-Process | Where-Object { $_.WorkingSet64 -gt 1MB } | 
                            Sort-Object CPU -Descending | Select-Object -First 8 | 
                            Select-Object Id, ProcessName, @{Name="CPU_Sec"; Expression={[math]::Round($_.CPU, 2)}}, @{Name="RAM_MB"; Expression={[math]::Round($_.WorkingSet64 / 1MB, 2)}}

            $sysmain = Get-Service -Name "SysMain" -ErrorAction SilentlyContinue
            $sysmainStatus = if ($sysmain) { $sysmain.Status.ToString() } else { "No Trobat" }
            $sysmainStart = if ($sysmain) { $sysmain.StartType.ToString() } else { "No Trobat" }

            $os = Get-CimInstance Win32_OperatingSystem
            $totalRam = [math]::Round($os.TotalVisibleMemorySize / 1024, 2)
            $freeRam = [math]::Round($os.FreePhysicalMemory / 1024, 2)
            $usedRam = [math]::Round($totalRam - $freeRam, 2)
            $ramPercent = [math]::Round(($usedRam / $totalRam) * 100, 1)

            $payload = @{
                Timestamp = (Get-Date).ToString("yyyy-MM-dd HH:mm:ss")
                System = @{
                    OSName = $os.Caption
                    TotalRAM = $totalRam
                    UsedRAM = $usedRam
                    FreeRAM = $freeRam
                    RAMPercent = $ramPercent
                    SysMainStatus = $sysmainStatus
                    SysMainStartType = $sysmainStart
                }
                Processes = $topProcesses
            }

            $json = $payload | ConvertTo-Json -Depth 4
            $buffer = [System.Text.Encoding]::UTF8.GetBytes($json)
            $response.ContentType = "application/json; charset=utf-8"
            $response.ContentLength64 = $buffer.Length
            $response.OutputStream.Write($buffer, 0, $buffer.Length)
        } else {
            $response.StatusCode = 404
        }
        $response.Close()
    }
} finally {
    $listener.Stop()
}
