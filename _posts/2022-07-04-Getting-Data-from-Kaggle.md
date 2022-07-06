# Getting Data from Kaggle

## Installation

Ensure you have Python 3 and the package manager `pip` installed.

Run the following command to access the Kaggle API using the command line:

`pip install kaggle` (You may need to do `pip install --user kaggle` on Mac/Linux.  This is recommended if problems come up during the installation process.) Installations done through the root user (i.e. `sudo pip install kaggle`) will not work correctly unless you understand what you're doing.  Even then, they still might not work.  User installs are strongly recommended in the case of permissions errors.

You can now use the `kaggle` command as shown in the examples below.

If you run into a `kaggle: command not found` error, ensure that your python binaries are on your path.  You can see where `kaggle` is installed by doing `pip uninstall kaggle` and seeing where the binary is.  For a local user install on Linux, the default location is `~/.local/bin`.  On Windows, the default location is `$PYTHON_HOME/Scripts`.

IMPORTANT: We do not offer Python 2 support.  Please ensure that you are using Python 3 before reporting any issues.

## API credentials

To use the Kaggle API, sign up for a Kaggle account at https://www.kaggle.com. Then go to the 'Account' tab of your user profile (`https://www.kaggle.com/<username>/account`) and select 'Create API Token'. This will trigger the download of `kaggle.json`, a file containing your API credentials. Place this file in the location `~/.kaggle/kaggle.json` (on Windows in the location `C:\Users\<Windows-username>\.kaggle\kaggle.json` - you can check the exact location, sans drive, with `echo %HOMEPATH%`). You can define a shell environment variable `KAGGLE_CONFIG_DIR` to change this location to `$KAGGLE_CONFIG_DIR/kaggle.json` (on Windows it will be `%KAGGLE_CONFIG_DIR%\kaggle.json`).

For your security, ensure that other users of your computer do not have read access to your credentials. On Unix-based systems you can do this with the following command: 

`chmod 600 ~/.kaggle/kaggle.json`

You can also choose to export your Kaggle username and token to the environment:

```shell
export KAGGLE_USERNAME=<username>
export KAGGLE_KEY=xxxxxxxxxxxxxx
```

In addition, you can export any other configuration value that normally would be in
the `$HOME/.kaggle/kaggle.json` in the format "`KAGGLE_<VARIABLE>`" (note uppercase).  
For example, if the file had the variable "proxy" you would export `KAGGLE_PROXY`
and it would be discovered by the client.

## Downloading Dataset
The following code methods can be useful to download the the dataset from Kaggle based on teh competition name.

#### run_cmd Method
This method creates a simple way to run system commands directly from python using the `subprocess` module.

#### download_kaggle_dataset method

Use this method to configure your Kaggle credentials created in the above step and download the dataset based on the name of the competition. If the `kaggle.json` file isn't in the correct location, you can pass it's lcoation. 

If you are using Google colab, you can easily have the kaggle config file saved on your drive. The method will connect to Google Drive and configure the environment based on the `kaggle.json` file that it finds on Google Drive.

```python
def run_cmd(cmd, verbose=False, *args, **kwargs) -> None:
    """
    Run system command as a subprocess
    """
    import subprocess

    process = subprocess.Popen(cmd,
                               stdout=subprocess.PIPE,
                               stderr=subprocess.PIPE,
                               text=True,
                               shell=True)
    std_out, std_err = process.communicate()
    if verbose:
        print(std_out.strip(), std_err)


def configure_kaggle(config_file: str=""):
    """
    Configure Kaggle CLI

    Args:
        config_file (str): Kaggle config JSON file path
    """    
    import sys
    from google.colab import drive

    if not config_file:
        kaggle_config_file = "MyDrive/.kaggle/kaggle.json"

    KAGGLE_CONFIG = "~/.kaggle/kaggle.json"
    dir_path = os.path.dirname(os.path.realpath(KAGGLE_CONFIG))
    run_cmd(f"mkdir -p {dir_path}")    
    if "MyDrive" in config_file:
        print("Found a Goolge Drive path, connecting to Google drive")
        drive.mount("/content/google_drive", force_remount=True)
        run_cmd(f"cp /content/google_drive/{config_file} {KAGGLE_CONFIG}")
        drive.flush_and_unmount()
    else:
        run_cmd(f"cp {config_file} {KAGGLE_CONFIG}")
     
    print("Configured Kaggle")


def download_kaggle_dataset(competition: str, kaggle_config_file: str="") -> None:
    """
    Download kaggle competition datasets.
    Args:
        competition (str): Kaggle comptetiton name
        kaggle_config_file (str): Kaggle config json file path
                                    Default is empty which would be internally be 
                                    treated as google drive location under "MyDrive/.kaggle/kaggle.json"
    """        
    if not kaggle_config_file:
        kaggle_config_file = "MyDrive/.kaggle/kaggle.json"
    config_kaggle(kaggle_config_file, is_file_in_drive)
    print(f"Downloading dataset for the competetion \"{competition}\" from Kaggle")
    run_cmd(f"kaggle competitions download -c {competition}")
    print(f"Extracting \"{competition}\" dataset")
    run_cmd(f"unzip -q {competition}.zip -d {competition}")

download_kaggle_dataset("us-patent-phrase-to-phrase-matching")
```