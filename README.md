# BlazorAppContainerLinux

This project deploys a blazor server application inside of a linux container. The container is then published to an Azure App Service that supports linux containers.

The docker file was created from a reference from this documentation:
- https://docs.microsoft.com/en-us/azure/app-service/configure-custom-container?pivots=container-linux#detect-https-session
- https://github.com/Azure-App-Service/dotnetcore (upgraded to .netcore 3.1)

Notes:
To enable remote SSH from the app service, you can use this preview cli command:

`az webapp create-remote-connection --subscription <subscription-id> --resource-group <resource-group-name> -n <app-name> &`

https://docs.microsoft.com/en-us/azure/app-service/configure-linux-open-ssh-session#open-ssh-session-from-remote-shell

# Known Issues

- When enabling a remote connection with app service, the remote debugging flag must be turned off in the app service config. When trying to use the tools in Visual Studio / VSCode to remotely debug, they will automatically re-enable this flag which will prevent a remote SSH connection.
- Connection Manager in Visual Studio 2019 establishes an SSH connection with the remote container and then the connection is terminated. Syslog has been enabled in the docker file and SSH logs (Debug level 3) can be found here (using SSH console in portal or putty):
  - /var/log/daemon.log
  - /var/log/syslog
  
# Current Work Around
VS Code has an extension called "remote ssh (preview v0.62.0)" (https://code.visualstudio.com/docs/remote/remote-overview) that can be configured to sucessfully connect and remotely debug.
