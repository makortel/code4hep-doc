Spack recipes for Code4hep
==========================

For official spack documentation see https://spack.readthedocs.io/en/latest/index.html

Recipe to get started
---------------------

Set up Spack environment
~~~~~~~~~~~~~~~~~~~~~~~~

In a directory where you want to set up Spack's package installations

.. code-block:: sh

   # One time
   git clone --depth=2 https://github.com/spack/spack.git

   # Needed every time to use Spack and the Spack packages
   source spack/share/spack/setup-env.sh

To avoid filling up ``/tmp`` when building Spack packages

.. code-block:: sh

   cat <<EOF > $HOME/.spack/config.yaml
   config:
     build_stage:
       - $spack/var/spack/stage      # Builds inside the Spack installation
   EOF

On LPC nodes it is also useful to move the ``$HOME/.spack`` out from the home directory to save space there

.. code-block:: sh

   mv $HOME/.spack $HOME/nobackup/
   ln -s $HOME/nobackup/.spack $HOME

Check what compilers are available for Spack

.. code-block:: sh

   spack list gcc

If newest GCC is older than 14, you can install it via Spack itself

.. code-block:: sh

   spack install -j 8 gcc@14

Generally Spack packages do not seem to propagate the C++ standard to their dependencies, so it is best to set that globally by editing spack package configuration with

.. code-block:: sh

   spack config edit packages

and adding

.. code-block:: sh

   packages:
     all:
       variants: cxxstd=20

there.

Set up Code4hep via Spack
~~~~~~~~~~~~~~~~~~~~~~~~~

Add Code4hep package repository

.. code-block:: sh

   spack repo add https://github.com/code4hep/c4h-spack-packages

Install ROOT

.. code-block:: sh

   spack install root@6.36.12 cxxstd=20 davix=false dcache=false gdml=false geom=false gsl=false math=false minuit=false opengl=false roofit=false tmva-cpu=false tmva-cudnn=false unuran=false vdt=false webgui=false x=false

ROOT would be installed via dependencies of Stitched/Code4hep as well, but installing it explicitly allows easy control of ROOT's dependencies. The default installation options would lead to ~160 dependencies to be installed.

Install Stitched
~~~~~~~~~~~~~~~~

.. code-block:: sh

   # install
   spack install stitched %gcc@14

   # Load Stitched into current environment
   spack load stitched
   source $(which stitched_env.sh)

   # Test that it runs
   cmsRun -h

Useful Spack commands
---------------------

List all available packages

.. code-block:: sh

   spack list

Show installed packages

.. code-block:: sh

   spack find

Uninstall a package

.. code-block:: sh

   spack uninstall <package name>

Stitched development with Spack packages
-----------------------------------------

See `Spack's tutorial <https://spack-tutorial.readthedocs.io/en/latest/tutorial_developer_workflows.html>`_ for more information.

Setup
~~~~~

.. code-block:: sh

   git clone https://github.com/code4hep/stitched-alpha2.git
   mkdir dev-env
   cd dev-env
   spack env create -d .
   spack env activate . # or 'spacktivate .'
   spack develop --path ../stitched-alpha2 stitched@2026-05-28 %gcc@14
   spack concretize


   # Unload
   spack env deactivate # or 'despacktivate'

The following command will show the packages that have been added to the development environment but not yet installed (among other information)

.. code-block:: sh

   spack find

To remove a package from the development environment

.. code-block:: sh

   spack remove <package>

Build Stitched manually with dependencies from Spack
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: sh

   # Activate previously made dev-env if not yet active
   spack env activate dev-env

   # Install Stitched's dependencies
   spack install --only dependencies stitched@2026-06-15

   # Start a new shell in a build environment
   spack build-env stitched -- bash

.. code-block:: sh

   cd stitched-alpha2

   # Configure the project into 'build' directory
   cmake -S . -B build

   # Build
   cmake --build build -j 8

Use by building Stitched package
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: sh

   # Activate previously made dev-env if not yet active
   spack env activate dev-env

   # Add CMake and compiler packages to the environment
   spack add gcc@14 cmake

   # Add Stitched package to the environment
   spack add stitched

   # Install all packages in the environment
   spack install

You can keep the logs by doing the installation as

.. code-block:: sh

   spack install --keep-stage


You can update or reinstall an already installed package without having to uninstall it first with

.. code-block:: sh

   spack install --overwrite stitched

Spack package development
--------------------------

.. code-block:: sh

   git clone https://github.com/code4hep/c4h-spack-packages

   # Replace Code4hep package repository with the local clone
   spack repo rm code4hep
   spack repo add c4h-spack-packages/spack_repo/code4hep

Follow earlier instructions to install Stitched. You don't necessarily need the development environment for just developing the packages.

If any ``package.py`` have been changed, need to

.. code-block:: sh

   spack concretize -f
