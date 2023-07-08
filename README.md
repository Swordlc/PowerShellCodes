# PowerShellCodes
# Shift+右键选择文件夹目录启动Windows PowerShell

# 遍历文件创建文件夹
# 遍历每个.mp4文件，创建相应名字的文件夹，将其移动到相应名字的文件夹中
# 将filter删除即不限文件格式，或在filter中规定新的文件格式如.mp3 .json等

Get-ChildItem -Filter *.mp4 | ForEach-Object {
    $name = $_.Name.Replace(".mp4","")  
    New-Item -ItemType Directory -Name $name
    Move-Item -Path $_.FullName -Destination $name  
}

# 遍历视频文件信息读取到文档
# 遍历目录下包括每个子文件夹的.mp4文件，使用FFmpeg实现读取帧宽度、帧高度，并写入一个响应的新.yml键值对文件中
Get-ChildItem -Recurse | ForEach-Object {
if($_.PSIsContainer){ 
    cd $_.FullName
    Get-ChildItem -Filter *.mp4 | ForEach-Object {
    $filename = $_.BaseName 
    
    $width_str = ffprobe -v error -select_streams v:0 -show_entries stream=width -of default=noprint_wrappers=1 "$($_.FullName)" 
    $match = [regex]::match($width_str, '\d+')    
    $width = $match.Value    
    
    $height_str = ffprobe -v error -select_streams v:0 -show_entries stream=height -of default=noprint_wrappers=1 "$($_.FullName)"
    $match = [regex]::match($height_str, '\d+')
    $height = $match.Value

    New-Item "$($_.Directory.FullName)\asset.yml" -ItemType File 
    
 @"   
VideoName: $($filename).mp4
Size:
  X: $width
  Y: $height
Position:
  X: 0
  Y: 0
"@ | Out-File "$($_.Directory.FullName)\asset.yml"    
}
}
}
