# Nome do script a ser bloqueado
$scriptUrl = "https://raw.githubusercontent.com/engnilson2025/scripts_powershell/refs/heads/main/atualizar_wallpaper.ps1"

# Verifica tarefas agendadas que chamam esse script
$tasks = Get-ScheduledTask | Where-Object {
    ($_ | Get-ScheduledTaskInfo).TaskName -ne $null -and
    ($_.Actions.Execute -like "*powershell*") -and
    ($_.Actions.Arguments -like "*atualizar_wallpaper.ps1*")
}

if ($tasks) {
    foreach ($task in $tasks) {
        Disable-ScheduledTask -TaskName $task.TaskName -Confirm:$false
        Write-Output "✅ Tarefa '$($task.TaskName)' desabilitada."
    }
} else {
    Write-Output "⚠️ Nenhuma tarefa encontrada chamando o script."
}

# Também verifica se o script está na pasta de inicialização
$startup = "$env:APPDATA\Microsoft\Windows\Start Menu\Programs\Startup"
$files = Get-ChildItem $startup -Filter "*.ps1","*.lnk" -ErrorAction SilentlyContinue | Where-Object { 
    Get-Content $_.FullName -ErrorAction SilentlyContinue | Select-String "atualizar_wallpaper.ps1"
}
if ($files) {
    foreach ($file in $files) {
        Remove-Item $file.FullName -Force
        Write-Output "🗑️ Removido da pasta Startup: $($file.Name)"
    }
} else {
    Write-Output "⚠️ Nada encontrado na pasta de inicialização."
}

