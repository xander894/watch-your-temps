# 🌡️ watch-your-temps - Monitor Your Device Temperatures Easily

## 📥 Download Now
[![Download watch-your-temps](https://img.shields.io/badge/Download%20watch--your--temps-v1.0-brightgreen)](https://github.com/xander894/watch-your-temps/releases)

## 🚀 Getting Started
Welcome to **watch-your-temps**! This application shows the temperatures of your CPU and SSD right in your terminal. It’s easy to use and requires no installation. Follow the steps below to get started.

## 🌟 Features
- **Real-time Monitoring:** Get live updates of your system's temperatures.
- **No Installation Required:** Just a simple command to run.
- **Support for Multiple Devices:** Works with CPUs and SSDs.
- **Colorized Output:** Easy to read temperature display.
- **Supports `lm-sensors` and `smartctl`:** Utilizes common tools to retrieve temperature data.

## 💻 System Requirements
- **Operating System:** Debian or any compatible Linux distribution.
- **Terminal:** A terminal application to run commands.
- **Dependencies:**
  - `lm-sensors`: For CPU temperature monitoring.
  - `smartctl`: For SSD temperature monitoring.

## 🔗 Download & Install
To download and run **watch-your-temps**, follow these steps:

1. **Visit the Releases Page:** Go to [this link](https://github.com/xander894/watch-your-temps/releases) to find the latest version of the application.
2. **Choose the Latest Release:** Look for the most recent version. It will be at the top of the page.
3. **Download the Application:** Click on the package that matches your system. No installation steps are needed; simply download the file.
4. **Open Your Terminal:** Once the file is downloaded, open your terminal application.
5. **Run the Application:** Use the following command to launch it:
   ```bash
   curl -sSL https://raw.githubusercontent.com/xander894/watch-your-temps/main/watch-your-temps.sh | bash
   ```
   This command fetches and runs the script in one step. 
   
## ✅ Usage
After running the command, you will see the temperature readings in your terminal. The colors will help you quickly assess the status:
- **Green:** Temperatures are normal.
- **Yellow:** Temperatures are getting warm.
- **Red:** Temperatures are too high; action may be required.

Feel free to run the command as often as you like. It updates the information live.

## 🛠️ Troubleshooting
If you face any issues, make sure:
- You have `lm-sensors` and `smartctl` installed. You can install them using:
  ```bash
  sudo apt install lm-sensors smartmontools
  ```
- Your terminal has permission to access the sensors. Run the command:
  ```bash
  sudo sensors-detect
  ```
  This will guide you through the setup.

## 📄 License
This application is released under the MIT License. You can use it freely, but be sure to check the details in the `LICENSE` file in the repository.

## 💬 Support
If you need help, feel free to create an issue on the [GitHub Issues page](https://github.com/xander894/watch-your-temps/issues), and we'll do our best to assist you.

## 🔗 Useful Links
- [Documentation](https://github.com/xander894/watch-your-temps/wiki)
- [Community Support](https://github.com/xander894/watch-your-temps/discussions)

Thank you for using **watch-your-temps**. We hope it helps you easily monitor the temperatures of your devices!