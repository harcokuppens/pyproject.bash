# pyproject.bash
A wrapper script around Python's venv and pip-tools to EASE Python project maintenance, by
automate setting up the Python virtual environment with the EXACT same dependencies at 
any location you want. (using a developer's lockfile)

## Intro:

### Setup project:

Given a python project with a modern 'pyproject.toml' configuration we can easily setup
the project for portable deployment:

 1) first add 'pyproject.bash' to your project

         wget https://raw.githubusercontent.com/harcokuppens/pyproject.bash/main/pyproject.bash
         git add pyproject.bash
 
 2) run setup:

         source pyproject.bash setup

    you now have an active python project: you have an **active venv** with **all needed dependencies installed**.
 
 3) store the generated lockfile.txt in your repository and push to remote

         git add lockfile.txt
         git push

### Reactivate project:

On a different developer machine you could now just clone your project's git repository, and then run:

       source pyproject.bash reactivate

Now a **virtual environment** with the **same python version** and the **exact same dependencies** is created
using the **lockfile in your repository**. You have now the exact same developer installation as on 
your other developer machine where you did the setup.
  
     
### Project info:

With the 'info' command you can easily get the status of your current project environment. Below
is an example:

        $ source pyproject.bash  info
        
        project venv: ACTIVE
        project config:  pyproject.toml context.txt
        
        venv folder: yes
        venv synced with LOCKFILE:  lockfile.mac.txt
        venv PYTHON VERSION: 3.8
        venv: activated
        
        python versions available on this system:
        
            python3.7
            python3.7m
            python3.8
            python3.8-intel64
            python3.9
            python3.10
            python3.10-intel64
            python3.11
            python3.12
            python3.12-intel64
        
        lockfiles with corresponding python versions:
        
            LOCKFILE                                                    PYTHON VERSION
            --------                                                    --------------
            lockfile.linux.txt                                          3.8
            lockfile.txt                                                3.9
            lockfile.mac.txt                                            3.8



## Usage details:

    $ source pyproject.bash

    USAGE: source project.bash setup [-l LOCKFILE] [-p PYTHONVERSION] [-f]
           source project.bash reactivate [-l LOCKFILE]
           source project.bash info|activate|clean
       
    Effortless and quickly setup of the exact same Python development project anywhere.

    A wrapper script around Python's venv and pip-tools to EASE Python project maintenance. 
    Using a developer's lockfile, stored in your repository, it automates setting up 
    the Python virtual environment with the same python version and EXACT same dependencies 
    at any location you want.
 
    How do you use it? Easy, first setup your project with the 'setup' command, 
    and commit the generate lockfile to your git repository. Then at any location you can
    just git clone the project, and execute the 'reactivate' command to reactive the project
    at that location. Done. 

    The 'setup' command determines the required dependencies from the config files,  
    and then locks these dependencies in a lockfile, so we can easily recreate the same 
    developer environment. Multiple lockfiles for different platforms are possible.

    The 'reactivate' command does automatically create and activate a virtual environment, 
    from a lockfile, where it automatically installs all needed dependencies.
 
    Using the 'info' command we can easily view the current project status.

    The wrapper script requires a modern 'pyproject.toml' project configuration setup, 
    from which it can reads the projects dependencies.

    On Windows machines you can install the required bash with 'Git for Windows' or 'WSL2'.

    Below we explain the commands in more detail using the wrapped venv, pip-compile, and 
    pip-sync tools. For more details about these tools see:

      * https://docs.python.org/3/library/venv.html
      * https://pip-tools.readthedocs.io

    The commands explained in more detail:
   
       setup      : Setup project env from config files pyproject.toml and context.txt(optional).
   
            Creates a lockfile from these config files (using pip-compile), then reactivates
            the project (see next command). The setup command locks the dependencies of the
            project to fixed versions in the lockfile. This lockfile allows us to reactivate
            the project with the exact same dependencies on other development locations. The
            lockfile is by default '$LOCKFILE' but can be changed with the -l option. If the
            lockfile already exists, then your are first asked permission to overwrite it.
            With the option '-f' you force overwrite. The default python version used is
            python3, which on this system is python3.12. 
            Most systems have python3 set to a reasonable version, and is therefore a
            reasonable default choice. However if your project requires a specific python
            version, you can require this version with the -p option. If you want to use
            the same python version as in an already existing lockfile, you can use the
            'info' command to find out which python version that lockfile uses. The
            'info' command also lists all available python versions on the system.

                
       reactivate : Reactivates an already setup project with an existing lockfile.
                
            Creates (if not yet exists) venv with python version in lockfile, activates venv,
            and syncs venv to lockfile (using pip-sync). Syncing venv means packages listed
            in the lockfile are installed, and packages installed not listed in the lockfile
            are uninstalled. The lockfile is by default '$LOCKFILE' but a different lockfile
            can be specified with the -l option.
                               
       activate   : Activates project in a (new) bash shell (requires venv up to date)
                
            Only activates venv in current bash shell; convenient for opening the project
            in a new bash shell. 
                
       clean      : Cleanup project virtual environment.
   
            Deactivates project and removes .venv, but keeps the lockfile(s). Using the
            lockfile the project can be reactivated.
                
       info       : Show project status.
   
            Lists details of current venv, lockfiles and python versions.               

    Normally a project gets 'setup' once, after which you store the lockfile in your project
    repository. Then on each new checkout of the project you only need to 'reactivate' it
    with the stored lockfile to get the exact same development setup.

    You only need to run 'setup' again when
      - you changed the project dependencies in pyproject.toml/context.txt or 
      - you changed platform: on run 'setup' if some python packages differ in dependencies
        for the change in platform. You should then keep a specific lockfile per platform.
        You can do this by using the '-l LOCKFILE' option to specify a platform specific
        lockfile. For example for macos arm64 we could use 'lockfile.macos-arm64.txt'.
      - When you change the python version of your project. Package versions can depend on
        python version. First run 'source project.bash clean' to remove the old '.venv'
        folder. Then run the 'setup' command with the new python version in the the '-p
        PYTHONVERSION' option.
     
    
    Because the '.venv' folder can be recreated it should not be include in your repository.
    You can even 'clean' the project from its '.venv' folder to reduce storage space for the
    project, and recreate it using the 'reactivate' command when you want to develop in the
    project again.

    If you only want to open a new bash shell for the project you can just use the 'activate'
    command.
        
    Your project may have some related projects on which your project depends, or your
    project is a dependency off. We call them 'context' projects, because these are in the
    context of your project. Often during development when making changes to your project you
    may need related changes in the context projects. Using editable installs python allows
    us to also install these context projects editable in your project. Because the editable
    installs are only needed during development we do not want to configure them in the
    pyproject.toml config. The latter file is used for building a production ready release
    package. Therefore we use a separate configfile 'context.txt' in which we configure the
    editable installs. The convention is to git clone a context project X into the context/X/
    subdir of your project, and the add a line '-e context/X' to the context.txt config file.
    Git cloning the project must be done manually, but after that the wrapper script can do
    the editable install automatically for you. The editable install also is added to the
    projects lockfile, so if we later need to reactivate the project it will automatically
    use the editable install instead of a normal install.

    The context.txt and the lockfile are only used during development. 
    For production we only need pyproject.toml. 
 
