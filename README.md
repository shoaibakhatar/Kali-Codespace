Here’s an updated **README.md** (your original steps + a **GUI/Desktop (XFCE via noVNC)** section for Codespaces).

````md
![Kali-Codespace](https://g.top4top.io/p_3536nulms0.png)

# Kali Codespace (Kali Linux in GitHub Codespaces)

This tutorial shows how to set up a Kali Linux container in GitHub Codespaces for pentesting using terminal and **web-based GUI tools (noVNC)**.

---

## 1. Update & Upgrade System

```bash
sudo apt update && sudo apt upgrade -y
````

---

## 2. Create Project Directory (Optional)

```bash
mkdir -p ~/kali-codespace
cd ~/kali-codespace
```

---

## 3. Create Dockerfile

```bash
echo 'FROM kalilinux/kali-rolling

RUN apt update && apt install -y \
    git curl wget python3 python3-pip \
    ca-certificates build-essential sudo \
    && apt clean

RUN useradd -ms /bin/bash rosemary && echo "rosemary:kali" | chpasswd && adduser rosemary sudo
RUN echo "rosemary ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

USER rosemary
WORKDIR /home/rosemary

CMD ["/bin/bash"]' > Dockerfile
```

**Notes**

* Default user: `rosemary`
* Password for user `rosemary`: `kali`
  (This sets the *user* password, not the root password.)

---

## 4. Create docker-compose.yml

> ✅ Updated to forward GUI port (noVNC) from container → Codespaces

```bash
echo 'version: "3.8"

services:
  kali:
    build: .
    container_name: kali-cs
    tty: true
    stdin_open: true
    ports:
      - "6080:6080"   # noVNC (GUI in browser)
    volumes:
      - kali-data:/home/

volumes:
  kali-data:' > docker-compose.yml
```

**Notes**

* Volume `kali-data` stores persistent data under `/home/`.

---

## 5. Build Container

```bash
docker compose build
```

---

## 6. Run Container

```bash
docker compose up -d
```

---

## 7. Access Kali Container (CLI)

```bash
docker exec -it kali-cs /bin/bash
```

---

# ✅ GUI / Desktop in Browser (XFCE via noVNC)

GitHub Codespaces doesn’t show Linux GUIs directly, but you can run a lightweight desktop (XFCE) inside the container and access it in your browser using **noVNC**.

## A) Install GUI packages (inside container)

Run these **inside** the container:

```bash
sudo apt update
sudo apt install -y xfce4 xfce4-goodies dbus-x11 \
  tigervnc-standalone-server tigervnc-common novnc websockify
```

## B) Set VNC password (first time only)

```bash
vncpasswd
```

## C) Configure VNC startup (XFCE)

```bash
mkdir -p ~/.vnc
cat > ~/.vnc/xstartup << 'EOF'
#!/bin/sh
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
startxfce4 &
EOF
chmod +x ~/.vnc/xstartup
```

## D) Start VNC server (display :1 → port 5901)

```bash
vncserver :1 -geometry 1366x768 -depth 24
```

## E) Start noVNC (web GUI on port 6080)

```bash
/usr/share/novnc/utils/novnc_proxy --listen 6080 --vnc localhost:5901
```

Kali’s own documentation uses the same `novnc_proxy` helper under `/usr/share/novnc/`.

## F) Open the GUI in Codespaces

1. In Codespaces, open the **Ports** tab.
2. Find port **6080** (or add it).
3. Click **Open in Browser**.

You should see the Kali XFCE desktop in your browser.

### Stop / restart GUI services

**Stop VNC:**

```bash
vncserver -kill :1
```

**Stop noVNC:** press `Ctrl + C` in the terminal running `novnc_proxy`.

---

## Notes

* Codespaces containers stop after being idle.
* Save important files in the volume (`kali-data`) or the repository.
* For safety, keep forwarded ports **Private** in the Codespaces “Ports” tab unless you intentionally need public sharing.

---

## References

* Kali Linux Docker Images (kali-rolling)
* GitHub Codespaces Documentation
* Kali “Kali in the Browser (noVNC)” documentation (uses `novnc_proxy` under `/usr/share/novnc/`)

```

If you want, I can also adjust it so **GUI auto-starts** when the container starts (using an entrypoint/supervisord), but the README above keeps it simple and manual.
::contentReference[oaicite:0]{index=0}
```
