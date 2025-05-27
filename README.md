<aside>
## Install Jupyterhub & Cloudflared Documentation
</aside>

- Jupyterhub Server
    1. Update & Upgrade apt
        
        ```bash
        sudo apt update && sudo apt upgrade -y
        ```
        
    2. Install Library
        
        ```bash
        sudo apt install python3 python3-pip python3-venv npm -y
        ```
        
    3. Create environment
        
        ```bash
        sudo mkdir -p /opt/jupyterhub_venv
        sudo python3 -m venv /opt/jupyterhub_venv
        source /opt/jupyterhub_venv/bin/activate
        ```
        
    4. Install Jupyter Notebook & Jupyter Hub
        
        ```bash
        pip install jupyterhub jupyterlab notebook
        ```
        
    5. Install HTTP Proxy
        
        ```bash
        sudo npm install -g configurable-http-proxy
        ```
        
    6. Setting Config.py
        - Step by Step
            1. Create folder
                
                ```bash
                sudo mkdir -p /etc/jupyterhub
                ```
                
            2. Generate file
                
                ```bash
                jupyterhub --generate-config -f /etc/jupyterhub/jupyterhub_config.py
                ```
                
            3. Configuration file
                
                ```bash
                sudo nano /etc/jupyterhub/jupyterhub_config.py
                ```
                
                เอาเครื่องหมาย `#` ออก (uncomment) และแก้ไขบรรทัดต่อไปนี้ (ถ้าจำเป็น)
                :`c.JupyterHub.bind_url = 'http://:8000'` (เพื่อให้ JupyterHub รับการเชื่อมต่อที่ port 8000 ทุก IP Address ของเครื่อง)
                
            4. Setting file ( find name like this and enable it )
                
                ```bash
                c.Authenticator.allow_all = True
                ```
                
    7. Add user account on Ubuntu
        
        ```bash
        sudo adduser <your-user>
        ```
        
    8. Create Service file
        
        ```bash
        sudo nano /etc/systemd/system/jupyterhub.service
        ```
        
    9. Paste Configuration
        
        ```bash
        [Unit]
            Description=JupyterHub
            After=network.target
        
            [Service]
            User=root  # หมายเหตุ: เพื่อความง่ายในการ bind port และจัดการ process user อื่นๆ (ดูข้อควรระวัง)
            Environment="PATH=/opt/jupyterhub_venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ExecStart=/opt/jupyterhub_venv/bin/jupyterhub -f /etc/jupyterhub/jupyterhub_config.py
            WorkingDirectory=/etc/jupyterhub/ # JupyterHub จะสร้างไฟล์ jupyterhub.sqlite ที่นี่
            Restart=always
            RestartSec=10
        
            [Install]
            WantedBy=multi-user.target
        ```
        
    10. Starting Service
        
        ```bash
        sudo systemctl daemon-reload
        sudo systemctl enable jupyterhub.service
        sudo systemctl start jupyterhub.service
        systemctl status jupyterhub.service
        ```
        
    
- Cloudflared
    1. Get Repository
        
        ```bash
        echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared $(lsb_release -cs) main' | sudo tee /etc/apt/sources.list.d/cloudflared.list
        ```
        
    2. Update & Upgrade apt
        
        ```bash
        sudo apt update
        sudo apt install cloudflared -y
        ```
        
    3. Login cloudflared
        
        ```bash
        cloudflared login
        ```
        
        - Set Config on Cloudflared Website
            1. Insert domain name
            2. Setting on domain dashboard
                1. Turn off DNS Security
                2. Copy name server (NS) from domain dashboard to cloudflared
                3. Copy name server (NS) from cloudflared to domain dashboard
            3. Waiting for activate cloudflared & Authorized it
            4. If Authorized successful go to 4.
    4. Create Tunnel
        
        ```bash
        cloudflared tunnel create <your-tunnel-name>
        ```
        
    5. Create Tunnel
        
        ```bash
        cloudflared tunnel run --url http://localhost:8000 <your-tunnel-name-or-uuid>
        ```
        
        - If you want to set cloudflared to automate enable server while using computer are opening.
            1. move config
                
                ```bash
                sudo mkdir -p /etc/cloudflared/
                sudo mv ~/.cloudflared/config.yml /etc/cloudflared/config.yml
                sudo mv ~/.cloudflared/f233fb20-af2d-4a87-9940-543658d3e5e3.json /etc/cloudflared/
                ```
                
            2. **change path in `/etc/cloudflared/config.yml`:**
                
                ```bash
                sudo nano /etc/cloudflared/config.yml
                
                #in credentials-file change it to
                #credentials-file: /etc/cloudflared/<your-uuid>
                ```
                
            3. Install Service
                
                ```bash
                sudo cloudflared service install
                ```
                
            4. Start & Enable service
                
                ```bash
                sudo systemctl start cloudflared
                sudo systemctl enable cloudflared
                ```
                
            5. Check Status
                
                ```bash
                sudo systemctl status cloudflared
                ```
