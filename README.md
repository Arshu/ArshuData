# ArshuData

Deploy in BareMetal (With GeminiAI Help), MicroVM (fly.io) and Unikernel (Unikraft)

The Declarative UI Source of the Application with the propritory binary is available in this repository

#### In OvhCloud Baremetal (https://ovharshudata.roottns.com/)

```
    Baremetal Server is hosted in France with the following specification
        OS - Ubuntu Server 24.04 "Noble Numbat" LTS
        CPU - Intel Xeon E5-1620v2 - 4c/8t - 3.7 GHz/3.9 GHz
        RAM - 32 GB ECC 1333 MHz DDR3
        Disks - 1×120 GB SSD SATA

    Auto Shutdown On Idle Web Application is .NET 9 ASP.NET Core NativeAot Compiled started using SystemD Socket Activation
    Deployment using Ngnix Reverse Proxy using Stream Block to forward to SystemD Socket Activated Web Application with TLS Termination
```

```
    Cost per Month : 16 SGD for above server with Unlimited Bandwidth (Restricted to 300 Mbps outgoing/incoming)

    ***Limited Liability due to fixed total cost but deployment requires skill, pending automation in UI***
```

#### In Fly.io MicroVM ((https://flyarshudata.arshucs.com/))

```
    Running on Two Machines in iad (Ashburn, Virginia (US)), ams (Amsterdam, Netherlands)
        OS : Alpine with docker image size less then 40 MB
        CPU : shared-cpu-1x
        RAM : 256 MB Memory
        Disks : 1GB Volume

    Auto Shutdown on Idle Web Application is .NET 9 ASP.NET Core NativeAot Compiled and packaged as Alpine oci image started using overmind with a procfile
```

```
    Cost per Month (Usage Based). Maximum $1.94 (iad), Maximum $2.02 (ams), with 1GB Volume ($0.15) and Bandwidth cost based on Usage

    ***Unlimited Liability due to Bandwidth Charges, but deployment is easy and can be automated in UI***
```

#### In Unikraft Unikernel ((https://uniarshudata.arshucs.com/))

```
    Running on One Instance in Fra (France)
        OS : Unikernel
        CPU : 1 vcpu
        RAM : 512
        Disks : 100 MB Volume
```

    Auto Shutdown on Idle/Unikraft Shutdown Web Application is .NET 9 ASP.NET Core NativeAot Compiled and packaged as Unikraft oci image started using Unikraft base ELF Loader
```
    Cost per Month : 19 USD for 16 concurrent, 128 instances with Unlimited Bandwidth

    ***Limited Liability due to fixed total cost, but deployment is easy with limitations, pending automation in UI***
```

# Disclaimer/Status

Experimental deployment in Usage Based Infrastructure for building a Franchise App Hosting framework using Arshu Html + Json Full Stack Declarative Framework (React Like in C# backend with Htmx Like frontend)

To deploy into Usage based Infrastructure with reduced switching cost, the framework is being modified/optimizated and hence expected to have issues even though most of the framework features are complete and stable.

Currently Manual deployment is possible and Auto Update has been scripted to make Redeployment Possible. 

UI based deployment for fly.io is complete using fly.io App and Machine Rest api with IP GraphQL api and using oci registry api to directly deploy repository images to fly registry. (https://pagelah.com/Hub/Config)

Pending UI Based deployment for Unikraft and OVH (Fully UI may not be possible without Manual)

# Manual Deployment to fly.io

## Prerequisite in Windows

Fly Client Install (https://fly.io/docs/flyctl/install/) after Registration and have login credentials.

Podman Download and install (https://podman.io/)

Execute the run.bat and configure the intial admin user id and password in the web ui and rerun to ensure that the wwwassets folder is updated

Execute the below commands in Windows Terminal

#### Step 1 - Build the Dockerfile
```
    cd to the ArshuData root folder of the repository after git clone https://github.com/Arshu/ArshuData.git
    podman build -f DockerfileFly -t <fly-appname>:latest    
```

#### Step 2 - Create the Fly App
```
    flyctl apps create --machines --name <fly-appname> --org <fly-orgname>
    flyctl ips allocate-v6 --app <fly-appname>
    flyctl ips allocate-v4 --shared --app <fly-appname>
```

#### Step 3 - Push the Docker Image to Fly Registry 
```
    podman tag <fly-appname>:latest registry.fly.io/<fly-appname>:latest
    podman push <fly-appname>:latest registry.fly.io/<fly-appname>:latest
    Alternate : flyctl deploy --app <fly-appname> --image registry.fly.io/<fly-appname>:latest
    Alternate : flyctl deploy --dockerfile DockerfileFly --build-only --local-only --push --image-label latest -a <fly-appname>
```

#### Step 4 - Create a Volume and Machine with Attached Volume
```
    fly volumes create data_<fly-appname>_iad_1 --region iad -a <fly-appname> //Note the Volume ID <fly-volumeid>
    echo app = "<fly-appname>" > fly.toml

    flyctl machine run registry.fly.io/<fly-appname>:latest --name <fly-appname>_iad_1 --port 443:8080/tcp:http:tls --env INFRA="FLY" --app <fly-appname> --region iad --config fly.toml --volume <fly-volumeid>:/app/wwwroot/App_Data --metadata fly_platform_version=v2

    Subsequent Update (Request Machine ID)
    flyctl machine update <fly-machineid> --image registry.fly.io/<fly-appname>:latest --port --port 443:8080/tcp:http:tls --env INFRA="FLY" --app <fly-appname> --yes --restart no --skip-health-checks --detach     

    Alternate : You can create a machine without Volume, then the data will not be persisted
```
