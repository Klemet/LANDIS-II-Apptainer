<p align="center"><img src="logo_Landis_Apptainer.svg" width="250"></p>

<h1 align="center">LANDIS-II Apptainer</h1>

A set of instructions to create and use an Ubuntu Apptainer containing the LANDIS-II model and its extensions, for use on almost any environment, including supercomputing clusters.

## 📦 What's an Apptainer ?

[Apptainer](https://apptainer.org/) (formerly called "Singularity") is a program that creates "containers", which are files containing an entire operating system (OS) like Ubuntu, along with every program or dependencies you might want to install in it.

You can imagine it as a portable, contained or encapsulated version of an OS that you can then deploy and use almost everywhere, even on supercomputing clusters.

As you will see below, and Apptainer can edit and modify files that are "outside" of the Apptainer. To do this, one simply "binds" specific folders that are outside the Apptainer when launching a command through the Apptainer. The Apptainer will then be able to consider these folders as inside of it, and will be able to act on them.

## 🌳 Why using an Apptainer with LANDIS-II ?

Apptainers have many advantages, and doubly so for people using the [LANDIS-II model](https://www.landis-ii.org/).

- They are easy to build (see below) and share.
- They make your results more reproducible, as you can provide the entire environment on which your LANDIS-II installation ran the simulations of your study; this includes any dependencies, versions of extensions, and so on. You can also include entire Python or R environments inside the Apptainer, which can be used to analyze your outputs from inside your Apptainer. 
- They will make your life much, much easier when dealing with several LANDIS-II environments (i.e. different set of extensions with different versions; especially with the upcoming switch to LANDIS-II v8 !)
- **They ensure that LANDIS-II will always have the right dependencies to function**. This is particularly crucial in many environments such as supercomputing clusters, which tend to restrain the programs/modules/packages/dependencies one can load inside your session. This can create hair-pulling scenarios where any update of the global environment in the clusters breaks your LANDIS-II installation, as it cannot access the right dependencies anymore. 

## 💾 How to download and use an already made Apptainer with LANDIS-II

This repository contains Apptainer files that I have already prepared with LANDIS-II. You can download them on the [release](https://github.com/Klemet/LANDIS-II-Apptainer/releases) page. **Be careful about the versions of the different extensions; be sure of what you're using !** If you need other versions, create an Apptainer yourself by following the instructions below; it's not too complicated, and it will save you time on the long run !

The Apptainer takes the form of a `.sif` file. The file is often fairly large (around 1GB minimum), as it contains an entire OS.

Once you've downloaded the file, the way to use your Apptainer depends on the OS you're using, and the environment in which you want to use the Apptainer. Here are instructions for usage on a Linux-based supercomputing cluster (as it's my own use case); see the [Apptainer documentation](https://apptainer.org/docs/user/latest/) if you need to use it on other OS or environments.

- Once you've downloaded the `.sif` Apptainer file, put it on the same computer/environment/session as your scenario files for LANDIS-II.
- Through the console (or in your bash script that launches your simulation), use `cd` to go into the folder containing your scenario file for LANDIS-II.
- Be sure to have the `apptainer` package available in your environment. If you're on a supercomputing cluster, be sure to load it; if on a personal computer, be sure to install it (see [here](https://apptainer.org/docs/user/latest/quick_start.html#installation) for instructions). Use `apptainer help` to be sure that `apptainer` is available.
- Then, you can launch a LANDIS-II simulation through the Apptainer in a single command. Here is an example with an explanation of what's going on :

```bash
apptainer exec -C -B {scenario_file_folder} ../ubuntuLANDIS_v1.sif /bin/sh -c "cd {scenario_file_folder} && dotnet /bin/LANDIS_Linux/build/Release/Landis.Console.dll scenario_main_file.txt"
```

- Here,
  - `apptainer` calls the `apptainer` package, which knows how to use your `.sif` file.
  - `exec` tells the `apptainer` package that we are going to run commands inside the Apptainer.
  - `-C` tells `apptainer` remove all of the folder binding that gets done automatically by apptainer. It will only bind the folders to the apptainer that we tell him to. The folder binding is essential : it's the process through which your Apptainer will become able to access files from outside the Apptainer file.
  - `-B ${scenario_file_folder}` binds the whole folder containing your scenario files. Replace `{scenario_file_folder}` with the **full, absolute path** to your scenario folder (e.g. `/home/Klemet/LANDIS-II_Simulations/Simulation1`). These files will become accessible inside the Apptainer at the same path (meaning at `/{scenario_file_folder}`). But the files are not "copied" in the Apptainer; the Apptainer will be editing the folder and files that are really outside of the Apptainer.
  - `../ubuntuLANDIS_v1.sif` is simply the location of the Apptainer file **relative to folder the console is in (normally, your simulation folder) when you launch the command**. In this example, the `.sif` Apptainer file is just outside the folder (hence using `..` to reach it).
  -  `/bin/sh -c "... && ..."` is used to give several commands at once to do inside the Apptainer. These two commands (`cd` and `donet`) are described just below.
  -  `cd {scenario_file_folder}` is a the first command we launch while inside the Apptainer : we simply use it to go inside the folder containing your LANDIS-II scenario files, which are accessible inside the Apptainer thanks to `-B ${scenario_file_folder}` that we used before, which "binded" them inside it.
  -  `dotnet /bin/LANDIS_Linux/build/Release/Landis.Console.dll scenario_main_file.txt` is the final command we launch to launch the simulation. We are inside the Apptainer; inside the folder with the LANDIS-II scenario files; and so we use the `dotnet` package (which is inside the Apptainer, since we're now launching commands from inside of it) to launch the LANDIS-II program through it's `.dll` file. The `.dll` is located inside the Apptainer I make in `/bin/LANDIS_Linux/build/Release/Landis.Console.dll`. Then, we give the argument needed for any LANDIS-II simulation, which the main scenario file (in this example `scenario_main_file.txt`) needed to start the simulation.
- You can give other commands to run inside the Apptainer after your simulation by adding `&& {Your other command}"`.

You can customize this command to the way you organize your files.

## ⚒️ Creating your own Apptainer file

Creating an Apptainer is not very difficult. It's done in four big steps :

- Initialize a _sandbox_ folder in which we will download and set up the OS files that will be inside the Apptainer `.sif` file in the end.
- Enter the _sandbox_ environment, which will give us access to a console that emulates an OS based on the files in the _sandbox_ folder.
- Install any program, packages and dependencies you want to have inside your final Apptainer file.
- Exit the _sandbox_, and create the Apptainer `.sif` file from the _sandbox_ folder.

Here, I will describe in details how I created a `.sif` file containing a Ubuntu 22.04 OS with LANDIS-II and many extensions installed from another Ubuntu 22.04 console. I used the Ubuntu console available on Windows (based on the Windows Subsystem for Linux or WLS) by using the [Ubuntu official "App" available on the Microsoft Store](https://apps.microsoft.com/detail/9pdxgncfsczv?hl=en-US&gl=US).

### Initializing the sandbox folder

- In your Ubuntu console, go into `/tmp`.
- Create the sandbox folder with `apptainer build --sandbox --fakeroot ubuntuLANDIS docker://ubuntu:22.04`
  - This will download Ubuntu 22.04 and prepare the sandbox folder in `/tmp/ubuntuLANDIS`

### Entering the sandbox environment

- While still in `/tmp`, use `apptainer shell --fakeroot --writable ubuntuLANDIS/` to enter the sandbox environment.
  - This will give you access to a console that is "inside" the sandbox environment. As such, any programs that you install or things that you modify from now on will be done inside the sandbox environment/folder that we created before, until you exit it with the command `exit`.
 
### Installing the programs you need in the sandbox environment

- Start by updating and upgrading any package in the default environment of your sandbox, and install the different programs you will need to install the rest (e.g. `wget`, `vim`, `pip`, `nano`, etc.)
```bash
apt update && apt upgrade
apt-get update && apt-get -y upgrade
apt-get install wget vim pip nano
```

#### Installing `dotnet`

We first need to install the dotnet SDK and runtime that you will need to compile and run LANDIS-II. The easiest is to use the `dotnet-install.sh` script that is made available by Microsoft for easy install (as dotnet is a library developed and managed by Microsoft).

- Use `mkdir /bin/.dotnet/` to create the directory where we will install the SDK.
- Use `wget https://dot.net/v1/dotnet-install.sh -O dotnet-install.sh` to download the dotnet install script.
- `chmod +x ./dotnet-install.sh` to make the script executable.
- `./dotnet-install.sh --channel 2.2 -InstallDir /bin/.dotnet/` to use the dotnet install script to install the SDK 2.2 (which is needed for LANDIS-II-v7) in the right folder
- And then `./dotnet-install.sh --channel 2.2 --runtime aspnetcore -InstallDir /bin/.dotnet/` to install the dotnet runtime 2.2.
- Also install `libssl` which is often needed with dotnet : `wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.0g-2ubuntu4_amd64.deb`, and then `dpkg -i libssl1.1_1.1.0g-2ubuntu4_amd64.deb`
- ⚠️ If you're installing LANDIS-II v8, it might require a different version of the SDK and runtime. See [this repository](https://github.com/LANDIS-II-Foundation/Core-Model-v8-LINUX) for more.
- To finish the procedure, and make the `dotnet` command easily available in the image, we need to edit the `PATH` global environment variable, that will point Ubuntu to the right folder to find our Dotnet install when calling the command `dotnet`. **But there is a little problem here** : From my experience, if you attempt to edit the `PATH` variable from inside the sandbox, it will edit the `PATH` variable outside the sandbox (from the Ubuntu installation we are using to create the Apptainer, but not the one inside in the Apptainer that we are editing here); see [here](https://apptainer.org/docs/user/main/environment_and_metadata.html) for more details.
  - To remedy this, start by exiting the sandbox (using the command `exit`); then, go in the `/tmp/ubuntuLANDIS` folder that contains the files of the sandbox environment
  - Then, edit the file `environnement` in the root of `ubuntuLANDIS` with a text editor (like `nano`), using a command like `nano environnement`.
  - Go to the end of the file in the editor, and add the two following lines :
    - `export DOTNET_ROOT=/bin/.dotnet`
    - `export PATH=$PATH:$DOTNET_ROOT:$DOTNET_ROOT/tools`
  - Save the file (`Ctrl` + `S` in `nano`) and exit it ( `Ctrl` + `X` in `nano`).
  - Go back to the `/tmp` folder, and use `apptainer shell --fakeroot --writable ubuntuLANDIS/` again to re-enter the sandbox environment.
- To check if everything is working, type `dotnet` in the console while inside the sandbox to see if it is properly installed. If it does not work, it must mean that the Ubuntu installation of the sandbox is not finding the `dotnet` installation, and that there must be an issue with the environment variables.
  - Also check if the right SDK and runtime are installed with `dotnet --list-sdks` and `dotnet --list-runtimes`.

#### Installing the Linux dependencies of LANDIS-II

Here are several libraries/packages that LANDIS-II will need in order to run on Linux

- `apt install libjpeg62`
- `apt install libpng16-16`
- `apt-get install gdal-bin`
- `apt-get install libgdal-dev`
- `export C_INCLUDE_PATH=/usr/include/gdal`
- `export CPLUS_INCLUDE_PATH=/usr/include/gdal`

#### Installing (compiling) LANDIS-II

This is the trickiest part. To run LANDIS-II on Linux, you need to compile it, along with any extension you want to use one by one.

It's not very difficult; but it's tedious, and there are often a couple of errors waiting for you along the way.

You will find the instructions for downloading the Core v7 and compiling extensions [here](https://github.com/LANDIS-II-Foundation/Core-Model-v7-LINUX); same for Core v8 [here](https://github.com/LANDIS-II-Foundation/Core-Model-v8-LINUX).

Here, I'll put instructions for v7; but things should be very similar for v8.

- Make a folder to install LANDIS-II in the sandbox with `mkdir /bin/LANDIS_Linux` and enter it with `cd /bin/LANDIS_Linux`
- Use `git` to clone the repository with the core of your choice : `git clone https://github.com/LANDIS-II-Foundation/Core-Model-v7-LINUX.git`
- Use `cd Core-Model-v7-LINUX/Tool-Console/src` to go into the folder with the source code of the core of LANDIS-II
- Use `dotnet build -c Release` to compile the core. Get used to this command; we'll use it afterwards again and again.
  - If you have issues or error when compiling the core in this way, please [go to the repository of the Core](https://github.com/LANDIS-II-Foundation/Core-Model-v7-LINUX) to indicate it as an issue.
- You should now have a `build` folder located in `Core-Model-v7-LINUX` that will contain the files needs to launch the core of LANDIS-II; especially the file `build/Release/Landis.Console.dll` mentioned in the section on using the Apptainer files.
- Go back to `/bin/LANDIS_Linux` (`cd /bin/LANDIS_Linux`), and end up by downloading the support libraries of LANDIS-II with the command `git clone https://github.com/LANDIS-II-Foundation/Support-Library-Dlls-v7.git`; move these libraries in the build/extension folder (`mv /bin/LANDIS_Linux/Support-Library-Dlls-v7* /bin/LANDIS_Linux/build/extensions`)

Then, you need to download and compile the extensions one by one. The process is similar for each and is described in the [repositories for the Linux version of the cores](https://github.com/LANDIS-II-Foundation/Core-Model-v7-LINUX).

- Go back to `/bin/LANDIS_Linux` (`cd /bin/LANDIS_Linux`)
- Clone the repository of the extension of your choice (e.g. `git clone https://github.com/LANDIS-II-Foundation/Extension-Biomass-Succession`).
  - **⚠️ This will download the latest version/git commit of the extension**; but this might not be what you want ! Especially if you want to compile the extensions for v7 of the Core. You can use the command `git checkout` while inside the downloaded folder to revert the files of the folder (especially the source code) back to a previous commit (check on Github with commit number you need to come back to a version of the repository where you'll have the version you want). 
- Modify the `.csproj` file (often located in the subfolder `src` of the folder that was created when cloning the repository, e.g. `/bin/LANDIS_Linux/Extension-Biomass-Succession/src`) according to the following instructions, with any way you would like (`nano` editor, or modifying the files directly through windows, or preparing the files in advance, etc.) :
![image](https://github.com/user-attachments/assets/a294424a-ee84-41b1-8b85-9d3b3bde548d)
- (1) Add the following line inside the `<PropertyGroup> ... </PropertyGroup>` tags : `<AppendTargetFrameworkToOutputPath>false</AppendTargetFrameworkToOutputPath>`
- (2) Add the following lines inside the `<Project> ... </Projects>` tags, that will tell dotnet where to place the compiled `.dll` :
 ```xml
  <PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Release|AnyCPU'">
	<OutputPath>..\build\extensions</OutputPath>
  </PropertyGroup>
```
- (3) In the `<HintPath>` tags of the `<ItemGroup>`, replace any path to the different dll with `..\..\build\extensions\{Name of the dll}.dll`. This will tell dotnet where to find the support dll we downloaded earlier, and which are needed for the compilation. Be careful here about the number of `..` in the path in order to point correctly to where the folder with the support dll is, **relative to the folder where the `.csproj` file is**.

Once you've done this, use `dotnet build` from inside the folder where the `.csproj` file is (e.g. do `cd /bin/LANDIS_Linux/Extension-Biomass-Succession/src` and then `dotnetbuild`). This test the compiling to see if there are any errors. Then, use `dotnet build -c Release` to finish the compiling. End up by checking if the `.dll` of the extension is correctly inside `/bin/LANDIS_Linux/build/extensions` (e.g. look for `Landis.Extension.Succession.Biomass-v5.dll` in the case of Biomass Succession v5).

**I highly recommend that you test if the extensions function properly right now, as you are still inside the sandbox environnement**. You better see if there are errors right now, as you're still able to correct them. Every extension folder cloned from Github often contains a set of test files; I recommend you run the test scenario that goes with them. To run it, go in the folder with the test scenario, and then use `dotnet /bin/LANDIS_Linux/build/Release/Landis.Console.dll {nameOftheScenarioMainFile}.txt`. LANDIS-II should launch the simulation with your test scenario properly at this point. If something doesn't work, be certain that the path to `Landis.Console.dll` is correct in your command. 

One last thing : I recommend you note the extensions you have compiled in this way and their versions. A good way to do this is by adding their information in the file `/bin/LANDIS_Linux/build/extensions/extensions.xml` :

![image](https://github.com/user-attachments/assets/d9d139fa-81b6-42fb-9ab9-12d12c4348be)

#### Installing any other program (Python, R, etc.) necessary for your simulations or analysis

If you plan on using Python, you can use the following :
- `apt install python3`
- `apt-get install python-is-python3`
- And then install any package you need with `pip`.

You can also install Anaconda or Miniconda, and then replicate an entire Python environment in your Apptainer.

Same thing goes with R.

### Finishing - creating the `.sif` file

- Exit the sandbox with the command `exit`.
- While being in `/tmp`, use `sudo apptainer build ubuntuLANDIS.sif ubuntuLANDIS` to create the `.sif` file.
- Once the file is made, I recommend using `apptainer overlay create --size 222 ubuntuLANDIS.sif`. This will create a small "editable layer" inside the `.sif` that can be useful to some programs, as the `.sif` is normally fixed (what's inside cannot be changed without re-creating a sandbox environment).

**Congratulations, you're done 🎉🎊 !**

To see how to use the Apptainer file to launch your LANDIS-II simulation, see the sections above.

### BONUS - Modifying or updating an Apptainer file

This is pretty simple :

- If you've kept the sandbox folder somewhere, just use the instructions above to enter it and modify it from the inside.
- If you haven't kept the sandbox folder, you can recreate one from a  `.sif` with the command `sudo apptainer build --sandbox <SANDBOX_DIR> <CONTAINER>.sif`; use `sudo apptainer shell --writable <SANDBOX_DIR>` to enter it and edit it, and then `sudo apptainer build <NEW_CONTAINER>.sif <SANDBOX_DIR>` to create your new `.sif` file.
