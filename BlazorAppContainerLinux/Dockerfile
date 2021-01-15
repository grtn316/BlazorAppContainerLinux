FROM mcr.microsoft.com/oryx/dotnetcore:3.1 as base
LABEL maintainer="Azure App Services Container Images <appsvc-images@microsoft.com>"

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        systemd \
        apt-utils \
        unzip \
        openssh-server \
        rsyslog \
        vim \
        curl \
        wget \
        tcptraceroute \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /defaulthome/hostingstart \
    && mkdir -p /home/LogFiles/ \
    && echo "root:Docker!" | chpasswd \
    && echo "cd /home" >> /etc/bash.bashrc

#COPY BlazorAppContainerLinux/bin.zip /tmp
#RUN unzip -q -o /tmp/bin.zip -d /defaulthome/hostingstart \
    #&& rm /tmp/bin.zip

COPY BlazorAppContainerLinux/init_container.sh /bin/
RUN chmod 755 /bin/init_container.sh

#COPY BlazorAppContainerLinux/hostingstart.html /defaulthome/hostingstart/wwwroot/

# configure startup
COPY BlazorAppContainerLinux/sshd_config /etc/ssh/
COPY BlazorAppContainerLinux/ssh_setup.sh /tmp
RUN mkdir -p /opt/startup \
   && chmod -R +x /opt/startup \
   && chmod -R +x /tmp/ssh_setup.sh \
   && (sleep 1;/tmp/ssh_setup.sh 2>&1 > /dev/null) \
   && rm -rf /tmp/*

ENV PORT 8080
ENV SSH_PORT 2222
EXPOSE 8080 2222

ENV WEBSITE_ROLE_INSTANCE_ID localRoleInstance
ENV WEBSITE_INSTANCE_ID localInstance
ENV PATH ${PATH}:/app
ENV ASPNETCORE_URLS=
ENV ASPNETCORE_FORWARDEDHEADERS_ENABLED=true

#WORKDIR /home/site/wwwroot

WORKDIR /app

FROM mcr.microsoft.com/dotnet/core/sdk:3.1-buster AS build
WORKDIR /src
COPY ["BlazorAppContainerLinux/BlazorAppContainerLinux.csproj", "BlazorAppContainerLinux/"]
RUN dotnet restore "BlazorAppContainerLinux/BlazorAppContainerLinux.csproj"
COPY . .
WORKDIR "/src/BlazorAppContainerLinux"
RUN dotnet build "BlazorAppContainerLinux.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "BlazorAppContainerLinux.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .

ENTRYPOINT ["/bin/init_container.sh"]

#COPY ["BlazorAppContainerLinux/sshd_config", "/etc/ssh/"]

#FROM mcr.microsoft.com/dotnet/core/aspnet:3.1-buster-slim AS base
#WORKDIR /app
#EXPOSE 80
#EXPOSE 443
#
#FROM mcr.microsoft.com/dotnet/core/sdk:3.1-buster AS build
#WORKDIR /src
#COPY ["BlazorAppContainerLinux/BlazorAppContainerLinux.csproj", "BlazorAppContainerLinux/"]
#RUN dotnet restore "BlazorAppContainerLinux/BlazorAppContainerLinux.csproj"
#COPY . .
#WORKDIR "/src/BlazorAppContainerLinux"
#RUN dotnet build "BlazorAppContainerLinux.csproj" -c Release -o /app/build
#
#FROM build AS publish
#RUN dotnet publish "BlazorAppContainerLinux.csproj" -c Release -o /app/publish
#
#FROM base AS final
#WORKDIR /app
#COPY --from=publish /app/publish .
#ENTRYPOINT ["dotnet", "BlazorAppContainerLinux.dll"]