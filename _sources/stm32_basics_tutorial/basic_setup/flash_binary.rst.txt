Flash Binary
============

.. contents:: Contents
   :depth: 2
   :local:


There are different methods to flash binary. We use **ST-Link** or **JLink** to flash binary to stm32 microcontrollers.

To install **stlink-tools**, **jlink** and **STM32CubeProgrammer**, see `installation <../../getting_started/installation.html>`_ and also `setup <../../getting_started/setup.html>`_.

1. Flash using ST-Link
----------------------

Connect your **ST-Link** to your **microcontroller**:

| - **SWCLK** of stlink to **SWCLK** of controller
| - **SWIO** of stlink to **SWIO** of controller
| - **GND** of stlink to **GND** of controller
  
If you want to provide power to controller using stlink, connect their ``3V3``.They are connected internally or through mini jumpers on the boards.

1.1. Flash using command
^^^^^^^^^^^^^^^^^^^^^^^^

Run this commnad in Terminal.

.. tabs::

   .. group-tab:: stlink-tools
      
      .. code-block:: bash
      
         # st-flash [--reset] write <build-directory/binaryfile.bin> <start-address>
         st-flash write build/BasicSetup.bin 0x8000000

      Use ``--reset`` option to reset afetr flash. Change binary file name to your program name.

   .. group-tab:: STM32CubeProgrammer

      .. code-block:: bash

         # STM32_Programmer_CLI -c port=SWD -w <build_directory>/<program_name>.bin 0x8000000
         STM32_Programmer_CLI -c port=SWD -w build/BasicSetup.bin 0x8000000

      Add ``-c port=SWD reset=SWrst`` at the end of line to reset after flash. Change binary file name to your program name.

.. attention::
   To **reset after flash**, make sure that **reset pin of st-link is connected to reset pin of microcontroller**. Otherwise, it will not work and give error. In discovery board, it is connected by default.

You do not need to write the long commnad every time, you can add them in ``Makefile``, ``CMakeLists.txt`` or create ``flash.sh``.
  

1.2. Makefile setup for flasing using ST-Link
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
- Add these at the bottom of **Makefile**.

  .. literalinclude:: makefiles/flash
     :language: makefile

  If you want to **reset after flash**, update to:

  .. literalinclude:: makefiles/flash_reset
     :language: makefile

- Flash binary.

  .. tabs::

     .. group-tab:: stlink-tools

        .. code-block:: bash
        
           make flash

     .. group-tab:: STM32CubeProgrammer

        .. code-block:: bash
        
           make pflash

1.2. CMakeLists.txt setup for flashing using ST-Link
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
- Add these lines at the bottom of **CMakeLists.txt**.
  
  .. code-block:: cmake
  
     # Flash target
     add_custom_target(flash
         COMMAND st-flash write ${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}.bin 0x8000000
         DEPENDS ${CMAKE_PROJECT_NAME}
         COMMENT "Flashing binary to microcontroller"
         WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
     )

     # Flash target
     add_custom_target(pflash
         COMMAND STM32_Programmer_CLI -c port=SWD -w ${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}.bin 0x8000000
         DEPENDS ${CMAKE_PROJECT_NAME}
         COMMENT "Flashing binary to microcontroller"
         WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
     )

- Flash binary by running this command from ``build`` folder after building ``binary``.

  .. tabs::

     .. group-tab:: stlink-tools

        .. code-block:: bash

           make flash

     .. group-tab:: STM32CubeProgrammer

        .. code-block:: bash
        
           make pflash

1.3. Bash script setup for flashing using ST-Link
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Create ``flash.sh`` at project folder.

  .. code-block:: bash
  
     touch flash.sh

- Add follwing lines.

  .. tabs::

     .. group-tab:: stlink-tools

        .. code-block:: sh
           :caption: flash.sh

           #!/bin/bash
   
           set +e
   
           st-flash write build/BasicSetup.bin 0x8000000

     .. group-tab:: STM32CubeProgrammer

        .. code-block:: sh
           :caption: flash.sh

           #!/bin/bash

           set +e
        
           STM32_Programmer_CLI -c port=SWD -w build/BasicSetup.bin 0x8000000

  Change binary file name to your program name.

- Make ``flash.sh`` executable.

  .. code-block:: bash
  
     chmod +x flash.sh

- Flash binary.

  .. code-block:: bash
  
     ./flash.sh

The most preferred method for **Makefile** is writting command in the **Makefile** and for **CMake** is creating ``flash.sh``.


2. Flash using JLink
--------------------

- Connect your **JLLink** to your **microcontroller**:

  - **SWCLK** of stlink to **SWCLK** of controller
  - **SWIO** of stlink to **SWIO** of controller
  - **GND** of stlink to **GND** of controller
  - **VREF** of jlink to **3V3** of controller

  Jlink does not provide power to controller. ``VREF`` is used to check the voltage logic only.

- Create ``flash.sh`` file at your project folder.

  .. code-block:: bash
     :caption: flash.sh

     #!/bin/bash

     set +e

     JLinkExe -if SWD -speed 4000 -autoconnect 1 -CommanderScript flash.jlink

- Create ``flash.jlink`` file at your project folder.

  .. code-block::
     :caption: flash.jlink
      
     device STM32F103C6
     r
     h
     loadbin build/<your_binary_filename>.bin, 0x08000000
     r
     g
     exit

  Change ``device name`` and ``binary filename``.

- Make ``flash.sh`` executable.

  .. code-block:: bash
  
     chmod +x flash.sh

- Flash binary.

  .. code-block:: bash
  
     ./flash.sh


There are also other methods to flash binary. You can use internal bootloader and USB to UART converter to flash binary using UART pins. To get into bootloader mode, you need to connect ``BOOT0`` to ``3V3`` and reset the controller. Then you can use ``stm32flash`` or ``STM32CubeProgrammer`` to flash binary. Some microcontrollers supports internal USB bootloader. If not, you can also create USB bootloader. Then, you can use ``dfu-util`` to flash binary using USB. Making USB bootloader is a bit complex and need to be done carefully. It also consumes some flash memory. So, it is not recommended for small memory microcontrollers.