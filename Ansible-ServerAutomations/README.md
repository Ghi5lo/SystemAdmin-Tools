# Ansible Windows Setup 🚀

This project is an Ansible playbook that automates the initial setup of a Windows server.

## 🔹 Features
- Enables WinRM for remote management  
- Creates an administrative user  
- Installs essential software (Notepad++, 7-Zip, Chrome, Git)  
- Configures the firewall (RDP)  
- Ensures Windows Defender is running 

## ⚙️ How to Use  
1. Clone the repository:  
   ```bash
   git clone https://github.com/Ghi5lo/SystemAdmin-Tools/Ansible-ServerAutomations.git
   cd Ansible-ServerAutomations
2. Edit inventory.ini with your Windows server's IP

3. Ensure WinRM is enabled on the server

4. Run the playbook:
    ```bash 
    ansible-playbook -i inventory.ini setup_windows.yml