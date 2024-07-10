# Migrating from Cherrytree to Joplin
# Note-Taking Apps 


Exporting from Cherrytree:
In Cherrytree select 
```
Export 
-> 
As Plain Text 
-> 
All the Tree (without making it Single File)
```
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

$folders = Get-ChildItem -Path $currentDirectory -Directory | Where-Object { $_.Name -match "ctb_TXT$" } | ForEach-Object { $_.Name }

$sourceDir = Join-Path -Path $currentDirectory -ChildPath ($folders -join ", ")
$destDir = "$($currentDirectory.Path)\NewExportedFolder"

#echo $sourceDir
#echo $destDir
#sleep 5

if (-not (Test-Path $destDir)) {
    New-Item -Path $destDir -ItemType Directory
}

$files = Get-ChildItem -Path "$sourceDir" -File

echo "files:"
echo $files
#sleep 10

foreach ($file in $files) {
    #echo $file
    
    # Get the file name without extension
    $fileName = $file.Name
    $baseName = [System.IO.Path]::GetFileNameWithoutExtension($fileName)
    
    # Update file name to get after -- 
    $fileName = $file.Name.Replace("_", " ")
    $pattern = "--\s*([^--]*)$"
    if ($fileName -match $pattern) {
        $fileName = $matches[1]
    }

    echo "fileName:"
    echo $fileName

    # Split the file name by "--"
    $parts = $baseName -split "--"
    #echo "Contain -- :"
    #echo $parts

    #sleep 1
    
    # If the file name contains "--", Create folder
    if ($parts.Length -gt 1) {
        $parentFolder = Join-Path -Path $destDir -ChildPath $parts[0]
        $currentFolder = $parentFolder
        
        # Create directory structure based on the parts
        for ($i = 1; $i -lt $parts.Length; $i++) {
            $currentFolder = Join-Path -Path $currentFolder -ChildPath $parts[$i]
            
            $currentFolder = $currentFolder.Replace("_", " ")
            if (-not (Test-Path $currentFolder)) {
                New-Item -Path $currentFolder -ItemType Directory
            }
        }

        # Copy the file to the final directory
        $destFilePath = Join-Path -Path $currentFolder -ChildPath $fileName
        #echo "destFilePath1:"
        #echo $destFilePath
        #sleep 1

        Copy-Item -Path $file.FullName -Destination $destFilePath
    } else {

        # For files without "--", move to the base folder
        $parentFolder = Join-Path -Path $destDir -ChildPath $parts[0]
        $parentFolder = $parentFolder.Replace("_", " ")
        if (-not (Test-Path $parentFolder)) {
            New-Item -Path $parentFolder -ItemType Directory
        }
        $destFilePath = Join-Path -Path $parentFolder -ChildPath $fileName
        #echo "destFilePath2:"
        #echo $destFilePath
        #sleep 1
        Copy-Item -Path $file.FullName -Destination $destFilePath
    }
}
```
```php
Press (Ctrl + S) -> then -> (Ctrl + W)
Then in cli run .\Ez.ps1
You Can now go to Joplin and Import As Text-Directory Our newly NewExportedFolder! :)



```

