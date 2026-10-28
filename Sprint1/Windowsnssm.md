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

```
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
```
<img width="338" height="51" alt="imatge" src="https://github.com/user-attachments/assets/b5b4fff2-6db8-43e5-8282-0ce8315f9412" />

<img width="479" height="199" alt="imatge" src="https://github.com/user-attachments/assets/d785545a-98cb-4b73-90f8-bcd53cae005d" />

- "Contingut arxiu .html"

```
<!DOCTYPE html>
<html lang="ca">
<head>
    <meta charset="UTF-8">
    <title>Windows Sentinel Dashboard</title>
    <style>
        body { font-family: Segoe UI, sans-serif; background: #0f172a; color: #f8fafc; padding: 20px; }
        .header { display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid #334155; padding-bottom: 10px; }
        .grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 20px; margin: 20px 0; }
        .card { background: #1e293b; border: 1px solid #334155; border-radius: 8px; padding: 15px; }
        .val { font-size: 2rem; font-weight: bold; color: #38bdf8; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; background: #1e293b; }
        th, td { text-align: left; padding: 10px; border-bottom: 1px solid #334155; }
        th { color: #38bdf8; background: #0f172a; }
        .red { color: #ef4444; }
        .green { color: #22c55e; }
    </style>
</head>
<body>
    <div class="header">
        <h1>🛡️ Windows Sentinel Dashboard</h1>
        <span id="status" style="background:#22c55e; color:#000; padding:5px 10px; border-radius:15px; font-weight:bold;">ONLINE</span>
    </div>
    <div class="grid">
        <div class="card">
            <h3>Ús de RAM</h3>
            <div id="ram" class="val">-- %</div>
            <p id="ram-info">--</p>
        </div>
        <div class="card">
            <h3>Estat Servei SysMain</h3>
            <div id="sysmain" class="val">--</div>
            <p id="sysmain-info">Tipus d'inici: --</p>
        </div>
        <div class="card">
            <h3>Sistema Operatiu</h3>
            <div id="os" style="font-weight:bold; margin-top:10px;">--</div>
            <p id="time" style="font-size:0.8rem; color:#94a3b8;">--</p>
        </div>
    </div>
    <h2>📊 Top Processos Actius</h2>
    <table>
        <thead><tr><th>PID</th><th>Nom Procés</th><th>CPU (s)</th><th>RAM (MB)</th></tr></thead>
        <tbody id="procs"></tbody>
    </table>
    <script>
        async function load() {
            try {
                const res = await fetch('/api/metrics');
                const data = await res.json();
                document.getElementById('ram').innerText = data.System.RAMPercent + '%';
                document.getElementById('ram-info').innerText = `${data.System.UsedRAM} MB / ${data.System.TotalRAM} MB`;
                const sm = document.getElementById('sysmain');
                sm.innerText = data.System.SysMainStatus;
                sm.className = (data.System.SysMainStatus === 'Stopped' || data.System.SysMainStartType === 'Disabled') ? 'val red' : 'val green';
                document.getElementById('sysmain-info').innerText = `Tipus d'inici: ${data.System.SysMainStartType}`;
                document.getElementById('os').innerText = data.System.OSName;
                document.getElementById('time').innerText = `Actualitzat: ${data.Timestamp}`;
                
                const tbody = document.getElementById('procs');
                tbody.innerHTML = '';
                data.Processes.forEach(p => {
                    tbody.innerHTML += `<tr><td>${p.Id}</td><td><b>${p.ProcessName}</b></td><td>${p.CPU_Sec || 0}s</td><td>${p.RAM_MB} MB</td></tr>`;
                });
            } catch(e) {}
        }
        load();
        setInterval(load, 3000);
    </script>
</body>
</html>
```

<img width="1920" height="1080" alt="imatge" src="https://github.com/user-attachments/assets/fe2fccaa-6e43-4a77-940e-b8be2f61fb1e" />

<img width="664" height="190" alt="imatge" src="https://github.com/user-attachments/assets/0fb9cc7e-e7f3-487f-b8b1-37ae9186a5a1" />

- Crear i configurar el servei Windows amb NSSM

<img width="425" height="232" alt="imatge" src="https://github.com/user-attachments/assets/2567f8fb-9a28-443e-a267-598e5de5f6eb" />

<img width="346" height="146" alt="imatge" src="https://github.com/user-attachments/assets/45f0cc71-a4eb-4c74-bec9-d2d8e57a011c" />

<img width="412" height="696" alt="imatge" src="https://github.com/user-attachments/assets/fc5c2341-79f4-4b56-9ab8-1254a489e106" />

<img width="423" height="130" alt="imatge" src="https://github.com/user-attachments/assets/b1d64e79-b57f-4b28-94c8-6da41b0256e9" />

- Després de reiniciar la màquina

<img width="446" height="152" alt="imatge" src="https://github.com/user-attachments/assets/8bdbd664-44f4-4b05-9d7a-acf4f659cacb" />

<img width="920" height="465" alt="imatge" src="https://github.com/user-attachments/assets/667222db-81cc-4c0f-a134-30212a8497c5" />
