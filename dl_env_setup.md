<h2>Objective</h2>

Provide a comprehensive guide for installing a Linux distro alongside Windows (dual boot) and setting up all necessary Nvidia drivers/libaries. It also follows a virtual enviroment paradigm for handling Python packages and creating environments with different independecies like different Python, CUDA, PyTorch/TensorFlow/NumPy/SciPy/matplotlib etc. versions.

<details>
    <summary>
<h2>Dual-boot Windows 11 and Ubuntu 22.04.3 LTS on your PC</h2>
    </summary>

  <details>
    <summary>
      <h3>Preliminary optional step: Disable Secure Boot</h3>
    </summary>
    
While UEFI Secure Boot is supported by Ubuntu, some issues still may rise like not be able to install some 3rd party kernel modules that are not signed. To disable Secure Boot, <b>first make sure to find your BitLocker recovery key</b>. Then follow these steps:
<ol>
  <li>Open the Start menu and open the power menu in the bottom right corner. Then, hold <b>Shift</b> on your keyboard and click <b>Restart</b>. After the PC restarts you will be taken to a blue screen with the following options:</li>
  <ul>
    <li>Continue</li>
    <li>Use a device</li>
    <li>Troubleshoot</li>
    <li>Turn off your PC</li>
  </ul>
  <li>Here, choose <b>Troubleshoot</b>, followed by <b>Advanced options</b>.</li>
  <li>Select <b>UEFI Firmware Settings</b> and then <b>Restart</b>.</li>
  <li>This will take you to your PC's BIOS settings, where you can turn off Secure Boot. Every BIOS is a little different, so you may have to look around. In the HP Omen laptop that we are using, navigate to the <b>BIOS Setup</b> and then from the upper left main menu select <b>Boot Options</b>. There, you can disable <b>Secure Boot</b> by moving the slider to the left. Lastly, click <b>Exit</b> and make sure that the <b>Save Changes and Exit</b> option is selected before choosing <b>Yes</b>.</li>
</ol>

</details>
</details>

<h2>Resources</h2>

  - [How to dual-boot Windows 11 and Linux on your PC](https://www.xda-developers.com/dual-boot-windows-11-linux/)
  - [Disable secure boot y/n?](https://askubuntu.com/questions/785120/disable-secure-boot-y-n)
