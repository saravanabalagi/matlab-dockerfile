# MATLAB Docker Image

This repo is forked from [matlab-dockerfile](https://github.com/mathworks-ref-arch/matlab-dockerfile) and has been modified to allow individual license files. 2021b has been tested to build and run successfully without issues.

## Before you begin
You need a valid MATLAB license
- Individual License (license.lic)
- Network License (network.lic)
- License Server (running at e.g. 27000@MyServerName)

You should be able to login to your Mathworks Account and be able to download MATLAB for linux.

## Requirements
You must perform these steps on a Linux platform.
Before starting, you must install the following on the client platform
- Docker
- Git

## Introduction
The following steps guide you through the process of creating a Docker container image that contains a Linux environment with a MATLAB installation. 
Use the container image as a scalable and reproducible method to deploy MATLAB in a variety of situations including clouds and clusters.

## Step 1. Clone this Repository
1. Clone this repository to your Linux client using 

    `git clone https://github.com/mathworks-ref-arch/matlab-dockerfile.git`
2. Inside the cloned repository, create a subdirectory named `matlab-install`

## Step 2. Choose MATLAB Installation Method
To install MATLAB into the container image, choose a MATLAB installation method. You can use MATLAB installation files or a MATLAB ISO image. 

### MATLAB Installation Files (Recommended)
To obtain the installation files, you must be an administrator for the license linked with your MathWorks account.
1. From the [MathWorks Downloads](https://www.mathworks.com/downloads/) page, select the desired version of MATLAB.
2. Download the Installer for Linux, extract and execute `install`.
3. [Download Products Without Installation](https://www.mathworks.com/help/install/ug/download-only.html).
    1. Log in to your account
    2. Accept EULA
    3. Click Advanced and select `Download Product without Installation`
    4. Specify a temporary download location.
    5. Select the products you want to install in the container image.
    6. Confirm your selections and complete the download. 
4. Move the downloaded folder to `matlab-install` subdirectory in this repo. Make sure you have the installation files (such as `install`, `license_agreement.txt`) directly within `matlab-install`.

### MATLAB ISO (Alternative Method)
1. From the [MathWorks Downloads](https://www.mathworks.com/downloads/) page, select the desired version of MATLAB.
2. Under the Related Links heading, click the link to get the ISO image for the chosen MATLAB version. 
3. Download the ISO image for the Linux.
4. Extract the ISO into the `matlab-install` subdirectory of the cloned repository.

## Step 3. Obtain the License File and File Installation Key
1. Log in to your [MathWorks account](https://www.mathworks.com/login). Select the license you wish to use with the container.
2. Select the Install and Activate tab. Select the link “Activate to Retrieve License File”.
3. Click the download link under the Get License File heading. 
4. Select the appropriate MATLAB version and click Continue.
5. At the prompt “Is the software installed?” select “No” and click Continue.
6. Download the `license.lic` and place it in the root directory.
7. Copy the File Installation Key into a safe location.

## Step 4. Define Installation Parameters
1. Make a copy of the file `installer_input.txt` in the `matlab-install` folder. Move the copy up one directory level, into the root directory of the cloned repository.
2. Rename the file to `matlab_installer_input.txt`.
3. Open `matlab_installer_input.txt` in a text editor and edit the following sections:
    - `fileInstallationKey` Paste your File Installation Key and uncomment the line.
    - `agreeToLicense` Set the value to yes and uncomment the line.
    - Specify products to install. Uncomment the line `product.MATLAB` to install MATLAB. Uncomment the corresponding line for each additional product you want to install. If you are not licensed to use a product, uncommenting the line does not install the product in the container. Your File installation Key identifies the products you can install.
4.	*(Optional)* Specify required dependencies in the `Dockerfile`. Edit the `Dockerfile` and uncomment the corresponding line for each dependency you want to add. For more information, see [Optional Dependencies](#optional-dependencies).

## Step 5. Build Image
Use the `docker build` command to build the image, using ```.``` to specify this folder. Run the command from the root directory of the cloned repository. Use a command of the form:
```
docker build -t matlab:r2021b --build-arg MATLAB_RELEASE=R2021b --build-arg LICENSE_USERNAME=saravanabalagi
```
- `--build-arg MATLAB_RELEASE` should be in the form of `R20xxx`, refers to a MATLAB release you are trying to build.
- `--build-arg LICENSE_USERNAME` should be the username for the individual license file

### Network License and License Manager
If you would like to use a network license `network.lic`, alter the `Dockerfile` as follows:
```
# Uncomment this line
ADD network.lic /usr/local/MATLAB/$MATLAB_RELEASE/licenses/

# Comment this line
# ADD license.lic /usr/local/MATLAB/$MATLAB_RELEASE/licenses/
``` 

For more information, see [Use a License File to Build Image](#use-a-license-file-to-build-image).

Alternatively, you can specify the location of the network license manager using `--build-arg LICENSE_SERVER=27000@MyServerName`. Replace `27000@MyServerName` with the port and location of your license manager. Update the `Dockerfile` appropriately as below:
```
# Uncomment these lines
ARG LICENSE_SERVER
ENV MLM_LICENSE_FILE=$LICENSE_SERVER

# Comment these lines
# ADD network.lic /usr/local/MATLAB/$MATLAB_RELEASE/licenses/
# ADD license.lic /usr/local/MATLAB/$MATLAB_RELEASE/licenses/
```
### For older versions <= 2019b
To build a previous version of MATLAB, use the `Dockerfile.R2019b` instead of `Dockerfile`: 
```
docker build -f Dockerfile.R2019b -t matlab:r2019b --build-arg MATLAB_RELEASE=R2019b --build-arg LICENSE_USERNAME=saravanabalagi .
```

## Step 6. Run Container
Use the `docker run` command to run the container. Use a command of the form:
```
docker run -it --rm --network host matlab:r2020a
```
- `-it` option runs the container interactively.
- `--rm` option automatically removes the container on exit.
- `--network host` enables [host networking](https://docs.docker.com/network/host/) mode, only needed when you use an individual license

Any extra arguments after the container tag are passed directly as command line arguments to the MATLAB process inside the container. For example, the following command prints `hello world` in MATLAB and then exits.
```
docker run -it --rm --network host matlab:r2021b -r "disp('hello world');exit"
```

## Optional Dependencies
For some workflows and toolboxes, you must specify dependencies. You must do this if you want to do any of the following tasks.
- Install extended localization support for MATLAB
- Play media files from MATLAB
- Run a network license manager inside the container
- Generate code from Simulink
- Use mex functions with gcc, g++, or gfortran
- Use the MATLAB Engine API for C and Fortran
- Use the Polyspace 32-bit tcc compiler

Edit the `Dockerfile` and uncomment the relevant lines to install the dependencies.

## Use a License File to Build Image
If you have a `license.dat` file from your license administrator, you can use this file to provide the location of the license manager for the container image.
1. Open the `license.dat` file. Copy the `SERVER` line into a new text file. 
2. Beneath it, add `USE_SERVER`. The file should now look something like this:
```
SERVER Server1 0123abcd0123 12345
USE_SERVER
```
3. Save the new text file as `network.lic` in the root directory of the cloned repository.
4. Open the `Dockerfile`, and comment the line `ENV MLM_LICENSE_FILE`
5. Uncomment the line `ADD network.lic /usr/local/MATLAB/$MATLAB_RELEASE/licenses/`
6. Run the docker build command without the `--build-arg LICENSE_SERVER=27000@MyServerName` option. Use a command of the form
```
docker build -t matlab:r2021b .
```
For more information about license files, see [What are the differences between the license.lic, license.dat, network.lic, and license_info.xml license files?](https://www.mathworks.com/matlabcentral/answers/116637-what-are-the-differences-between-the-license-lic-license-dat-network-lic-and-license_info-xml-lic)
