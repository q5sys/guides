**Boot Environments: Management and System Recovery 101**

Written by: Mike Reynolds

Boot Environments
*****************

What is a Boot Environment?
===========================

A Boot Environment (or BE) is a feature provided by the use of the ZFS
filesystem. A BE is a bootable |trueos| environment that includes
everything needed to boot the system. A BE is best described as a
snapshot of the current state of the system when the BE was created.
When |trueos| is installed, a new BE called :guilabel:`initial` is
created. This means that at any point in time a |trueos| system can
be reverted back to a fresh install by activating the
:guilabel:`initial` BE then booting in to it.

  .. note:: User files and directories are **not** included when
	    creating a boot environment. Only the directories required
	    for |trueos| to boot are included in the BE, :file:`/usr`,
	    :file:`/usr/local`, and :file:`/var`.

Creating and Managing Boot Environments.
========================================

|trueos| takes full advantage of boot environments. When a system
update is available, a new boot environment is automatically created as
a fail-safe prior to downloading and installing the update. This
providesan easy way to recover, or "rollback", an update should
anything go wrong during the process.

The |sysadm| Boot Environment Manager.
--------------------------------------

Creating boot environments are not limited to the automatically created
BEs by the update process. Boot environments can be created at any time
by users, not just by the system itself. |trueos| provides an easy to
use GUI to manage boot environments through the |sysadm| Control
Panel :sysclbk:`Boot Environments <boot-environment-manager>`.

Open the |sysadm| Control Panel and double-click Boot Environments as
shown in :numref:`Figure %s <SysAdmControlPanel>`.

.. _SysAdmControlPanel:

.. figure:: images/SysAdmControlPanel.png
   :scale: 100%

   SysAdm Control Panel

This will open the |sysadm| Boot-Up Configuration, which is used to
easily manage boot environments.
:numref:`Figure %s <Boot-UpConfiguration>` shows two BEs, the
:guilabel:`initial` BE created during the installation of |trueos|, and
:guilabel:`12.0-CURRENT-up-20180129_204321` the BE created prior to
running a system update, that is now currently in use indicated by
:guilabel:`NR` under the :guilabel:`Active` column.

.. _Boot-UpConfiguration:
.. figure:: images/Boot-UpConfiguration.png
   :scale: 100%

   Boot-Up Configuration

  .. note:: About the **NR** under the :guilabel:`Active` column;
	    **N** means active Now, **R** means active on Reboot, and
	    **NR** means active Now *and* on Reboot.

Creating New Boot Environments.
-------------------------------

There are two options available to create new BEs, :guilabel:`Create BE`
and :guilabel:`cloneBE`.

:guilabel:`Create BE`: This option will create a *new* BE using the
current state of the system. Any modifications done to **system** files
will be reflected when activating and booting into this BE. After
clicking the :guilabel:`Create BE` button, the New Boot Environment
naming dialog will open as shown in
:numref:`Figure %s <New Boot Environment>`. Enter a unique name for the
new BE. Attempting to name a BE the same as an existing BE will give an
error as seen in :numref:`Figure %s <Invalid Name>`.

.. _New Boot Environment:
.. figure:: images/CreateBE.png
   :scale: 100%

   New Boot Environment Name dialog

.. _Invalid Name:
.. figure:: images/InvalidName.png
   :scale: 100%

   Invalid Name Dialog

:guilabel:`cloneBE`: This option will clone the highlighted BE. Cloning
an existing BE will make an exact copy of the original BE, preserving
the original. Any changes made to the cloned BE will not modify the
original BE. This is useful for testing how modifications made to the
system will affect the system. If adverse results are found, activating
the original BE will bring the system back to its pre-modification
state. :numref:`Figure %s <ClonedBE>` shows a cloned BE. Notice that
the cloned BE is not the same size as the original BE. Only the changes
made since the original BE was created will make the BE grow or shrink
in size. For example, if a system binary that was 5GB was added to the
system, the cloned BE would increase by 5GB. This is yet another great
feature of boot environments.

.. _ClonedBE:
.. figure:: images/ClonedBE.png
   :scale: 100%

   Cloned Boot Environment

Deleting Boot Environments.
---------------------------

To delete a BE, select the BE from the list in the |sysadm| Boot
Environments GUI. Click the :guilabel:`Delete BE` button. The selected
BE will then be deleted. Note: There is **no confirmation** dialog after
clicking delete. Please be sure that the correct BE is selected
**before** clicking the :guilabel:`Delete BE` button.

.. TODO: Would it be worth adding a Note, or Tip here to reinforce the
   fact the there is now "are you sure" when deleting a BE?

Renaming a Boot Environment.
----------------------------

Renaming a boot environment is just that, changing the name from what
it currently is, to the name desired. For a simple example, a BE name
that contains the date '20180129' as shown in
:numref:`Figure %s <Rename BE>` would be out of date a year from when
the BE was created. It may be useful to rename the BE to reflect the
date to be more accurate.

.. _Rename BE:
.. figure:: images/RenameBE.png
   :scale: 100%

   Rename Boot Environment

.. TODO: Add Note about how the BE name affects the system management
   of boot environments, and how the default of 5 BEs are automagically
   cycled.
   I know I read this info somewhere, but decided to move on for now,
   and circle back once I found the info.

Mounting and Unmounting Boot Environments.
------------------------------------------

Any *currently inactive* BE can be mounted under :file:`/tmp` to allow
browsing/editing files in the mounted BE. To mount a BE, select the
desired BE, then click the :guilabel:`Mount BE` button. Once the BE has
finished mounting, the path to the mounted BE will be shown under the
:guilabel:`Mountpoint` column. Note: This is also an indication that a
BE is currently mounted. :numref:`Figure %s <Mounted BE>` shows an
example of a mounted BE.

.. _Mounted BE:
.. figure:: images/MountedBE.png
   :scale: 100%

   Mounted Boot Environment

One useful example mounting a BE allows would be to access
:file:`/etc/rc.conf` to add an environment variable, or remove an
erroneous entry. After mounting the desired BE, opening
:file:`/tmp/Name_of_theBE/etc/rc.conf` in a text editor would allow
edits to be made to the *BE's* :file:`rc.conf`. This would allow
enabling a kernel module, or changing the system's hostname prior
booting into the boot environment. Once finished with the BE, make sure
the BE is highlighted, then click the :guilabel:`Unmount BE` button to
unmount the BE.

Activating a Boot Environment.
------------------------------

To activate a boot environment, select the desired boot environment to
be made active, then click the :guilabel:`Activate BE` button. This will
set the activated BE to be used on the next boot of the system. As shown
in :numref:`Figure %s <Activate BE>` the :guilabel:`R` under the
:guilabel:`Active` column moves to the activated boot environment to
indicate that this is now the *active on **R**eboot* BE. Note, there is
no way to "deactivate" a boot environment. To "deactivate" a BE, simply
select the desired BE to be made active, and click the
:guilabel:`Activate BE` button.

.. _Activate BE:
.. figure:: images/ActivateBE.png
   :scale: 100%

   Activate Boot Environment

.. note:: For more information about the creation and management of BEs,
	  please refer to the |sysadm| Client Handbook about the
	  :sysclbk:`Boot Environment Manager <boot-environment-manager>`.

How to Recover an Unbootable System using Boot Environments
***********************************************************

Using the |sysadm| Boot Manager is not the only way to interact with or
set an alternate boot environment. BEs are a great option to recover an
unbootable system, be it after a failed upgrade, or even an erroneous
entry in :file`/etc/rc.conf`

Using the |trueos| Boot Menu to Select an Alternate BE.
=======================================================

After POST and the system makes it to the first |trueos| boot menu shown
in :numref:`Figure %s <First Boot Menu>`, select
:guilabel:`3 Select Boot Environment...".

.. _First Boot Menu:
.. figure:: images/BootMenu_First.png
   :scale: 100%

   First |trueos| Boot Menu

After entering the :guilabel:`Select Boot Environment...` menu, a list
of available boot environments on the system are presented as shown in
:numref:`Figure %s <Boot Environments Boot Menu>`. The list of BEs will
be the same as what is available via the |sysadm| Boot Environment
Manager GUI. Enter the number of the desired boot environment.

.. _Boot Environments Boot Menu:
.. figure:: images/BootMenus_BEs.png
   :scale: 100%

   Boot Environments Boot Menu

After selecting the desired boot environment, the choice will now be
reflected in the new boot options as shown in
:numref:`Figure %s <Boot Menu Alternate BE Selected>`. In this example,
the 12.0-CURRENT-up-20180129_CLONE BE was selected to boot and shows as
:guilabel:`1. Active: zfs:tank/ROOT/12.0-CURRENT-up-20180129_CLONE` to
indicate this BE is now the active BE that will be used to boot the
system.

.. TODO Add info about what option 2, "bootfs" indicates. It seems
   logical that this would be the original BE, but I was unable to
   confirm.

.. _Boot Menu Alternate BEs Menu:
.. figure:: images/BootMenus_AlternateBE.png
   :scale: 100%

   Boot Menu Showing the Alternate BE Selected

Enter option :guilabel:`3. Page: 1 of 1` to return to Page 1 of the boot
menu, then select option :guilabel:`1. Boot TrueOS [Enter]` as was shown
in :numref:`Figure %s <First Boot Menu>` (or hit guilabel:`Enter`) to
boot the newly selected BE. The system will now boot using the new
boot environment instead of the previous boot environment that would
not boot.

.. note:: Be sure to read the boot choices carefully and choose the
        correct option for "Select Boot Environment...". The number
        itself is irrelevant, what is important is the option itself.
        Be sure to choose the correct option

Some systems may have more or different options on the first boot menu
than what is shown in the example screenshot. For example, in
:numref:`Figure %s <Alternate Boot Menu>` the option to enter the boot
environment selection menu is actually
:guilabel:`7. Select Boot Environment...` As stated in the note, the
important thing is to enter the Boot Environment selection menu,
regardless of the actual number for the menu choice.

Booting into the Working Boot Environment.
------------------------------------------

After successfully booting the system into a working boot environment,
the |sysadm| Boot Environment Manager will confirm the system was booted
into the BE that was selected during boot as indicated by the
:guilabel:`N` (indicating active Now) under the Active column.

Now that the system is booted into a working boot environment, the
non-booting BE can be mounted as described in the
:ref:`Mounting and Unmounting Boot Environments.` section. This will
allow the unbootable boot environment to be investigated for issues or
simply discarded by deleting the BE as described in the
:ref:`Deleting Boot Environments` section.


Conclusion
**********

Boot environments are a powerful tool made simple to use with the tools
provided by |trueos|. The information in this guide only scratches the
surface of what is possible to do with boot environments.

.. TODO: Add links for further information. Advanced topics and more
   information about using BEs and possibly ZFS.
