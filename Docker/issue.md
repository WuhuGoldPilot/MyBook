<!--
 * @Author: silly
 * @Descripttion: In User Settings Edit
 * @LastEditTime: 2021-05-20 15:32:48
 * @Date: 2021-05-19 14:22:53
 * @LastEditors: silly
 * @FilePath: \notes\docker\issue.md
-->

# issue

## Docker cannot start on Windows

With Powershell:

Open Powershell as administrator
Launch command: & 'C:\Program Files\Docker\Docker\DockerCli.exe' -SwitchDaemon
OR, with cmd:

Open cmd as administrator
Launch command: "C:\Program Files\Docker\Docker\DockerCli.exe" -SwitchDaemon

## Docker: “no matching manifest for windows/amd64 in the manifest list entries”

Right click Docker icon in the Windows System Tray
Go to Settings
Daemon
Advanced
Set the "experimental": true
Restart Docker

## Error processing tar file(exit status 1): archive/tar: invalid tar header

docker save [image] -o file.tar followed by docker load -i file.tar