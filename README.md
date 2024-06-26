# Python project management with 'pyproject.bash'

Effortless and quickly setup the exact same Python development project anywhere. 

A wrapper script around Python's venv and pip-tools to EASE Python project maintenance. 
Using a developer's lockfile, stored in your repository, it automates setting up the Python 
virtual environment with the same python version and EXACT same dependencies at any location you want.

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

Look at an example project using pyproject.bash at: https://github.com/ev3dev-python-tools/thonny-ev3dev/blob/master/Developing.md#automated-setup
  
     
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


## Platform specific lockfiles

Python tries to give platform independent API. However, to provide this API on each specific platform it may require some different packages 
to be installed. If that is the case for your project then you could create a separate lockfile  per platform. Eg. a separate lockfile for linux,mac and windows.

If we have a lockfile for an platform, we then want to create a lockfile for another platform that deviates as little as possible. We can do this
easily by using the constraints option of `pip-compile`.
Using the constraints option of pip-compile we can  create a `lockfile.NEW.txt` for the NEW platform from the OLD platform's `lockfile.OLD.txt` by running the following command on the NEW platform:

         pip-compile  -c lockfile.OLD.txt   -o lockfile.NEW.txt   pyproject.toml context.txt 

All dependencies that both platforms use have the same versions, because of the constraint. Only for dependencies specific to the NEW platform the pip-compile will add new entries. Dependencies specific for the OLD platform are not added, because they are not needed for the new platform. Their entries in the constraint file are just ignored.

## IDE support

### Visual Studio Code

Vscode automatically recognizes the .venv folder in your main project and uses that python virtual environment.

To make vscode  see the git repositories of the context projects you need to add each folder of a context project to the workspace using the menu "File->Add Folder to Workspace...".
Then save the workspace as a `.code-workspace` file in your main project so that you can easily open it later again. Have a workspace setup also allows you to easily browse all projects in the "Explore" side menu  because they became root folders in the workspace view there.

### IntelliJ IDEA/Pycharm

The IDE doesn't always automatically recognize the  .venv folder in your main project, however via the Settings you can easily choose to use the python in that folder.

The IDE also doesn't automatically see the git repositories for your context projects, however at 
"Settings | Version Control | Directory Mappings" you can easily add them. 

## Internal documentation in pyproject.bash script:

    $ source pyproject.bash
    NAME
           source pyproject.bash - Setup the exact same Python development project anywhere.

    USAGE  
           source pyproject.bash setup [-l LOCKFILE] [-p PYTHONVERSION] [-f]
           source pyproject.bash reactivate [-l LOCKFILE]
           source pyproject.bash info|activate|clean
   
    DESCRIPTION   
        Effortless and quickly setup the exact same Python development project anywhere.

        A wrapper script around Python's venv and pip-tools to EASE Python project maintenance.
        Using a development's lockfile, stored in your repository, it automates setting up the
        Python virtual environment with the same python version and EXACT same dependencies at any
        location you want. 
 
        How do you use it? Easy, first setup your project with the 'setup' command. The 'setup'
        command determines the required dependencies from the config files, and then locks
        these dependencies in a lockfile, so we can easily recreate the same developer
        environment. Then at another location 'git clone' the project, and execute the
        'reactivate' command. The 'reactive' command then uses the info in the lockfile to
        reactive the project exactly the same! Using the 'info' command we can always
        easily view the current project status. And if you only want to open the project
        in a new bash shell you can just use the 'activate' command to just activate its
        sandboxed environment in the new shell.

        A more extensive tutorial is at https://github.com/harcokuppens/pyproject.bash .


    COMMANDS

     setup 
        Setup project from config files pyproject.toml and context.txt(optional).

        Creates a lockfile from these config files (using pip-compile), then reactivates
        the project (see next command). 

        The setup command locks the dependencies of the project to fixed versions in the
        lockfile. This lockfile allows us to reactivate the project with the exact same
        dependencies on other development locations. The lockfile is by default
        lockfile.txt but can be changed with the -l option. This make multiple lockfiles
        for different platforms possible. If the lockfile already exists, then your are
        first asked permission to overwrite it. With the option '-f' you can force
        overwrite.

        The default python version used is python3, which on this system is python3.12.
        Most systems have python3 set to a reasonable version, and is therefore a
        reasonable default choice. However if your project requires a specific python
        version, you can require this version with the -p option. If you want to use the
        same python version as in an already existing lockfile, you can use the 'info'
        command to find out which python version that lockfile uses. The 'info' command
        also lists all available python versions on the system.

        The pyproject.toml is the new standard config file to configure your python project.
        The context.txt config file is to configure your context projects dependencies
        during development. (see explanation below).

        The context.txt and the generated lockfile are only used during development.
               
      reactivate 
        Reactivates an already setup project with an existing lockfile.
    
        Creates (if not yet exists) venv with python version in lockfile, activates venv,
        and syncs venv to lockfile (using pip-sync). Syncing venv means packages listed
        in the lockfile are installed, and packages installed not listed in the lockfile
        are uninstalled. The lockfile is by default '$LOCKFILE' but a different lockfile
        can be specified with the -l option.
                           
      activate 
        Activates project in a (new) bash shell (requires venv up to date)
    
        Only activates venv in current bash shell; convenient for opening the project
        in a new bash shell. 
            
      clean
        Cleanup project virtual environment.

        Deactivates project and removes .venv, but keeps the lockfile(s). Using the
        lockfile the project can be reactivated.
            
      info 
        Show project status.

        Lists details of current venv, lockfiles and python versions.               

      help 
        Displays this documentation.



    PRACTICAL USAGE

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

  
    
    CONTEXT PROJECTS
    
        Your project may have some related projects on which your project depends, or your
        project is a dependency off. We call them 'context' projects, because these are in the
        context of your project. Often during development when making changes to your project
        you may need related changes in the context projects. Using editable installs python
        allows us to also install these context projects editable in your project. Because the
        editable installs are only needed during development we do not want to configure them
        in the pyproject.toml config. The latter file is used for building a production ready
        release package with 'python -m build'. Therefore we use a separate configfile
        'context.txt' in which we configure the editable installs purely used for development.

        The convention is to git clone a context project X into the context/X/ subdir of
        your project, and the add a line '-e context/X' to the context.txt config file.
        Also the main project is added with a line '-e .' to the context.txt config file.
        Editable install of the main project is required if your project uses the
        src-layout, because otherwise python files are in the src/ subdirectory won't be
        made available on the PYTHONPATH automatically. The editable install of your main
        project solves that!

        After git cloning the main project, then git cloning of the context projects into the
        context folder must be done manually or by a custom script.  
        Then the wrapper script can do all dependencies installs, including the editable
        installs automatically for you. The editable installs are also added to the
        project's lockfile, so if we later need to reactivate the project the editable installs
        are also done again.
        
        An example project using context projects:
        
            https://github.com/ev3dev-python-tools/thonny-ev3dev
 
        The context.txt and the lockfile are only used during development. 
        For production we only need pyproject.toml.
    

    ABOUT WRAPPER SCRIPT

        Why not use venv and pip-tools directly? The wrapper script saves you from many manual
        steps using these tools, where often you have to lookup in the documentation how you
        install and use these tools. The wrapper script automates these steps, and can directly
        placed in your repository for direct usage. And if you forgot something, you just run the
        script to lookup its internal documentation. Easy as that!

        This wrapper script is used only during development. In production we are more liberal in
        requiring which versions of a dependency we allow, otherwise we would get unresolvable
        conflicts with other packages requiring the same dependency. But in production we setup an
        virtual environment only containing the package you are developing. During development we
        want every dependency locked to a specific version. We first want to develop in that
        strict environment, and not be distracted by bugs caused by a different version of a
        dependency. In a later phase, when the software is more stable, we can test
        with a more liberal version dependency.

        The wrapper script requires a modern 'pyproject.toml' project configuration setup, 
        from which it can reads the projects dependencies. It also requires bash installed.
        On Windows machines you can install the required bash with 'Git for Windows' or 'WSL2'.
    
        For more details about the wrapped tools see:

          * https://docs.python.org/3/library/venv.html
          * https://pip-tools.readthedocs.io
     
