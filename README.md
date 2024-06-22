# Migrating from Cherrytree to Joplin
# Note-Taking Apps 


Exporting from Cherrytree:
In Cherrytree select Export -> As Plain Text -> All the Tree (without making it Single File).
Then Save the exported data in any folder, for example, "Folder Containing Exported Data".

PowerShell Automation:
Open PowerShell as Administrator and navigate to "Folder Containing Exported Data" as the working directory.

Run:
```
Set-ExecutionPolicy Bypass
notepad Ez.ps1
```
In the Notepad, paste the following automation script:
```powershell
$currentDirectory = Get-Location

$sourceDir = "$($currentDirectory.Path)\*.ctb_TXT"
$destDir = "$($currentDirectory.Path)\NewExportedFolder"

if (-not (Test-Path $destDir)) {
    New-Item -Path $destDir -ItemType Directory
}

$files = Get-ChildItem -Path $sourceDir -File

foreach ($file in $files) {
    # Get the file name without extension
    $fileName = $file.Name
    $baseName = [System.IO.Path]::GetFileNameWithoutExtension($fileName)
    
    # Split the file name by "--"
    $parts = $baseName -split "--"
    
    # If the file name contains "--"
    if ($parts.Length -gt 1) {
        $parentFolder = Join-Path -Path $destDir -ChildPath $parts[0]
        $currentFolder = $parentFolder
        
        # Create directory structure based on the parts
        for ($i = 1; $i -lt $parts.Length; $i++) {
            $currentFolder = Join-Path -Path $currentFolder -ChildPath $parts[$i]
            if (-not (Test-Path $currentFolder)) {
                New-Item -Path $currentFolder -ItemType Directory
            }
        }
        
        # Move the file to the final directory
        $destFilePath = Join-Path -Path $currentFolder -ChildPath $fileName
        Move-Item -Path $file.FullName -Destination $destFilePath
    } else {
        # For files without "--", move to the base folder
        $parentFolder = Join-Path -Path $destDir -ChildPath $parts[0]
        if (-not (Test-Path $parentFolder)) {
            New-Item -Path $parentFolder -ItemType Directory
        }
        $destFilePath = Join-Path -Path $parentFolder -ChildPath $fileName
        Move-Item -Path $file.FullName -Destination $destFilePath
    }
}
```

Press (Ctrl + S) -> then -> (Ctrl + W)
Then in cli run .\Ez.ps1
You Can now go to Joplin and Import Newly NewExportedFolder! :)


