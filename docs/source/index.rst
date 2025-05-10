Welcome to the Arise ATC'25 Artifact!
=========================================

Thank you for your interest in Arise! 
This document will guide you through the prerequisites and steps needed to set up and show the results described in our ATC'25 paper. 
Please feel free to reach out if you encounter any issues or have feedback.

Prerequisites
--------------

For optimal performance, we recommend running the full evaluation on a machine with the following specifications:

- **CPU**: 16 cores or more
- **Memory**: At least 32 GB
- **Disk Space**: 20 GB free

These requirements will ensure a smooth experience.

Getting Started (Kick-the-tire)
---------------

To begin, please download the artifact from Zenodo using this link: [`Artifact <https://doi.org/10.5281/zenodo.15375789>`_]. Once downloaded, extract the artifact archive, which may take around 10-30 minutes:

.. code-block:: console

  $ gunzip -c arise.tar.gz > arise.tar  # 10~30 minutes

Starting the Container
----------------------

Once you have imported the image, please use the following commands to start the container. This setup will grant the container the necessary permissions to run the evaluation smoothly.

.. code-block:: console

  $ docker load -i arise.tar
  $ docker run -it --name arise compiler_testing:arise /bin/bash

.. note::

  When the container starts successfully, a long hash string will appear on the terminal, indicating that the **Kick-the-tire** setup is complete.
  

Next Steps
----------

Once the container is up and running, you can follow the detailed instructions provided in the :doc:`/guide` section within the container. These instructions will guide you through using Arise.

Contents
--------

Below is an outline of the content provided for this artifact:

.. toctree::

   guide
   bug-report
