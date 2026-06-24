# WSL + Ubuntu + Docker + Containerlab Guide

**Prepared by: izzaldeen mansour**  
**إعداد: izzaldeen mansour**

هذا الدليل عبارة عن كتالوج عملي لتجهيز بيئة تدريبية لبرمجة الشبكات باستخدام WSL وUbuntu وDocker وContainerlab.  
الهدف أن يستطيع الطالب تشغيل بيئة Linux داخل Windows، تثبيت Docker، بناء شبكة افتراضية باستخدام Containerlab، ثم تجربة اتصال ورسائل بين `client1` و `client2` عبر `server`.

> **ملاحظة مهمة:** هذا الدليل لا يحتوي على أي IP خاص بجهاز معين، ولا يتضمن إعدادات Port Forwarding من Windows إلى WSL. كل العناوين المستخدمة تعليمية داخل اللاب فقط.

---

## Roadmap

| المرحلة | الهدف | الناتج المتوقع |
|---|---|---|
| 1 | تثبيت WSL وUbuntu | فتح Ubuntu Terminal داخل Windows |
| 2 | تجهيز Ubuntu | أوامر Linux الأساسية تعمل |
| 3 | تثبيت Docker | تشغيل Containers |
| 4 | تثبيت Containerlab | بناء شبكة من ملف YAML |
| 5 | إنشاء Topology | server + client1 + client2 |
| 6 | Deploy وفحص | ping ناجح بين النودات |
| 7 | مثال رسائل | client1 يرسل إلى client2 عبر server |

---

## 1. المتطلبات قبل البدء

- Windows 10 حديث أو Windows 11.
- صلاحية Administrator على الجهاز.
- اتصال إنترنت أثناء التثبيت.
- تفعيل Virtualization من UEFI/BIOS إذا كانت غير مفعلة.
- مساحة تخزين مناسبة للتجارب والصور، ويفضل 10GB أو أكثر.

لفحص Virtualization:

```text
Task Manager → Performance → CPU → Virtualization
```

إذا كانت `Disabled`، فعّلها من UEFI/BIOS حسب نوع الجهاز.

---

## 2. تثبيت WSL إذا لم يكن موجودًا

افتح PowerShell كمسؤول:

```text
Start → PowerShell → Right Click → Run as administrator
```

ثم نفذ:

```powershell
wsl --install
```

إذا طلب Windows إعادة تشغيل الجهاز، أعد التشغيل.

### 2.1 التأكد من حالة WSL

```powershell
wsl --status
wsl --list --verbose
```

إذا ظهر Ubuntu وكانت `VERSION = 2`، فالإعداد جيد.

### 2.2 تثبيت Ubuntu يدويًا عند الحاجة

```powershell
wsl --list --online
wsl --install -d Ubuntu
```

أو:

```powershell
wsl --install -d Ubuntu-24.04
```

يمكن أيضًا تثبيت Ubuntu من Microsoft Store بالبحث عن `Ubuntu` ثم الضغط على Install.

### 2.3 تشغيل Ubuntu لأول مرة

افتح Ubuntu من Start Menu. سيطلب منك:

```text
Enter new UNIX username: student
New password:
Retype new password:
```

اكتب username بحروف صغيرة وبدون مسافات.  
كلمة السر لا تظهر أثناء الكتابة، وهذا طبيعي في Linux.

### 2.4 جعل WSL2 هو الافتراضي

```powershell
wsl --set-default-version 2
wsl --set-version Ubuntu 2
```

إذا كان اسم التوزيعة مختلفًا، استخدم الاسم الظاهر في:

```powershell
wsl --list --verbose
```

---

## 3. تجهيز Ubuntu بالأدوات الأساسية

داخل Ubuntu نفذ:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y git curl wget nano vim iproute2 iputils-ping net-tools traceroute ca-certificates gnupg lsb-release
```

اختبر الأدوات:

```bash
git --version
curl --version
ip -V
ping -V
```

---

## 4. تثبيت Docker Engine داخل Ubuntu

Docker هو المسؤول عن تشغيل الـ Containers.  
Containerlab يستخدم Docker لتشغيل النودات وربطها.

### 4.1 إضافة مستودع Docker الرسمي

```bash
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

أضف Docker repository:

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

ثم:

```bash
sudo apt update
```

### 4.2 تثبيت Docker packages

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 4.3 تشغيل Docker

جرّب أولًا:

```bash
sudo systemctl enable --now docker
sudo systemctl status docker
```

إذا لم يعمل `systemctl` داخل WSL، استخدم:

```bash
sudo service docker start
sudo service docker status
```

### 4.4 استخدام Docker بدون sudo

```bash
sudo usermod -aG docker $USER
newgrp docker
```

يفضل إغلاق Ubuntu وفتحه من جديد بعد هذا الأمر.

### 4.5 اختبار Docker

```bash
docker --version
docker ps
docker run hello-world
```

إذا ظهرت رسالة `Hello from Docker`، فهذا يعني أن Docker يعمل.

---

## 5. تثبيت Containerlab

```bash
bash -c "$(curl -sL https://get.containerlab.dev)"
```

اختبر Containerlab:

```bash
containerlab version
```

أو:

```bash
clab version
```

---

## 6. مفاهيم أساسية

| المفهوم | المعنى |
|---|---|
| Image | قالب يحتوي نظام ومكتبات، مثل `alpine:latest` أو `python:3.12-alpine` |
| Container | نسخة شغالة من Image ويمكن اعتبارها جهازًا وهميًا خفيفًا |
| Node | أي طرف داخل الشبكة مثل client أو server أو router |
| Server | النود التي تقدم خدمة وتستقبل اتصالات |
| Client | النود التي تطلب خدمة أو ترسل بيانات للسيرفر |
| Link | وصلة افتراضية بين نودين |
| Topology | وصف الشبكة داخل ملف YAML |

---

## 7. إنشاء أول Topology بملف YAML

```bash
cd ~
mkdir -p network-project
cd network-project
nano lab.clab.yml
```

ضع داخل الملف:

```yaml
name: network-project

topology:
  nodes:
    client1:
      kind: linux
      image: alpine:latest
      exec:
        - ip addr add 10.0.1.11/24 dev eth1
        - ip link set eth1 up

    client2:
      kind: linux
      image: alpine:latest
      exec:
        - ip addr add 10.0.2.12/24 dev eth1
        - ip link set eth1 up

    server:
      kind: linux
      image: alpine:latest
      exec:
        - ip addr add 10.0.1.100/24 dev eth1
        - ip link set eth1 up
        - ip addr add 10.0.2.100/24 dev eth2
        - ip link set eth2 up

  links:
    - endpoints: ["client1:eth1", "server:eth1"]
    - endpoints: ["client2:eth1", "server:eth2"]
```

للحفظ في nano:

```text
Ctrl + O
Enter
Ctrl + X
```

---

## 8. Deploy وفحص النودات

```bash
sudo containerlab deploy -t lab.clab.yml
sudo containerlab inspect -t lab.clab.yml
docker ps
```

اختبر `client1`:

```bash
docker exec -it clab-network-project-client1 sh
ip addr
ping 10.0.1.100
exit
```

اختبر `client2`:

```bash
docker exec -it clab-network-project-client2 sh
ip addr
ping 10.0.2.100
exit
```

---

## 9. تجربة خدمة HTTP بسيطة على السيرفر

ادخل إلى server:

```bash
docker exec -it clab-network-project-server sh
```

داخل السيرفر:

```sh
apk update
apk add --no-cache python3
mkdir -p /www
echo "Hello from Containerlab server" > /www/index.html
python3 -m http.server 8080 --directory /www --bind 0.0.0.0
```

من Terminal آخر، اختبر من client1:

```bash
docker exec -it clab-network-project-client1 sh
wget -qO- http://10.0.1.100:8080
exit
```

واختبر من client2:

```bash
docker exec -it clab-network-project-client2 sh
wget -qO- http://10.0.2.100:8080
exit
```

---

## 10. مثال عملي: client1 يرسل رسالة إلى client2 عبر server

هذا المثال يبني Message Relay Server بلغة Python.

الفكرة:

```text
client2 يبقى متصلًا بالسيرفر
client1 يرسل رسالة إلى السيرفر
السيرفر يوجّه الرسالة إلى client2
```

### 10.1 تجهيز مجلد المثال

```bash
cd ~/network-project
sudo containerlab destroy -t lab.clab.yml --cleanup
mkdir -p message-demo/server message-demo/client
cd message-demo
```

### 10.2 ملف server.py

```bash
nano server/server.py
```

```python
import socket
import threading

HOST = "0.0.0.0"
PORT = 5000

clients = {}
lock = threading.Lock()

def handle_client(conn, addr):
    name = None

    try:
        first = conn.recv(1024).decode().strip()

        if not first.startswith("NAME:"):
            conn.close()
            return

        name = first.split(":", 1)[1]

        with lock:
            clients[name] = conn

        print(f"{name} connected from {addr}")

        while True:
            data = conn.recv(4096)

            if not data:
                break

            text = data.decode().strip()

            if text.startswith("TO:"):
                parts = text.split(":", 2)

                if len(parts) == 3:
                    target = parts[1]
                    message = parts[2]

                    with lock:
                        target_conn = clients.get(target)

                    if target_conn:
                        target_conn.sendall(f"FROM {name}: {message}\n".encode())
                        conn.sendall(f"DELIVERED to {target}\n".encode())
                    else:
                        conn.sendall(f"ERROR: {target} is not online\n".encode())

    finally:
        if name:
            with lock:
                clients.pop(name, None)

        conn.close()

def main():
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind((HOST, PORT))
    server.listen(10)

    print(f"Message server listening on {HOST}:{PORT}")

    while True:
        conn, addr = server.accept()
        threading.Thread(target=handle_client, args=(conn, addr), daemon=True).start()

if __name__ == "__main__":
    main()
```

### 10.3 ملف listener.py

```bash
nano client/listener.py
```

```python
import os
import socket
import time

NODE_NAME = os.environ.get("NODE_NAME", "client")
SERVER_IP = os.environ.get("SERVER_IP", "127.0.0.1")
SERVER_PORT = int(os.environ.get("SERVER_PORT", "5000"))

while True:
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((SERVER_IP, SERVER_PORT))
        s.sendall(f"NAME:{NODE_NAME}\n".encode())

        print(f"{NODE_NAME} connected to server {SERVER_IP}:{SERVER_PORT}")

        while True:
            data = s.recv(4096)

            if not data:
                break

            print(data.decode().strip(), flush=True)

    except Exception as e:
        print(f"Connection error: {e}")

    time.sleep(2)
```

### 10.4 ملف send_message.py

```bash
nano client/send_message.py
```

```python
import sys
import socket

if len(sys.argv) < 5:
    print("Usage: python send_message.py <server_ip> <sender_name> <target_name> <message>")
    sys.exit(1)

server_ip = sys.argv[1]
sender = sys.argv[2]
target = sys.argv[3]
message = " ".join(sys.argv[4:])

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((server_ip, 5000))

s.sendall(f"NAME:{sender}\n".encode())
s.sendall(f"TO:{target}:{message}\n".encode())

reply = s.recv(4096).decode().strip()
print(reply)

s.close()
```

### 10.5 Dockerfiles

ملف server:

```bash
nano server/Dockerfile
```

```dockerfile
FROM python:3.12-alpine

WORKDIR /app

COPY server.py .

EXPOSE 5000

CMD ["python", "server.py"]
```

ملف client:

```bash
nano client/Dockerfile
```

```dockerfile
FROM python:3.12-alpine

WORKDIR /app

COPY listener.py .
COPY send_message.py .

CMD ["python", "listener.py"]
```

### 10.6 بناء الصور

```bash
docker build -t msg-server:latest ./server
docker build -t msg-client:latest ./client
```

### 10.7 ملف lab-message.clab.yml

```bash
nano lab-message.clab.yml
```

```yaml
name: msg-lab

topology:
  nodes:
    client1:
      kind: linux
      image: msg-client:latest
      env:
        NODE_NAME: client1
        SERVER_IP: 10.0.1.100
      exec:
        - ip addr add 10.0.1.11/24 dev eth1
        - ip link set eth1 up

    client2:
      kind: linux
      image: msg-client:latest
      env:
        NODE_NAME: client2
        SERVER_IP: 10.0.2.100
      exec:
        - ip addr add 10.0.2.12/24 dev eth1
        - ip link set eth1 up

    server:
      kind: linux
      image: msg-server:latest
      exec:
        - ip addr add 10.0.1.100/24 dev eth1
        - ip link set eth1 up
        - ip addr add 10.0.2.100/24 dev eth2
        - ip link set eth2 up

  links:
    - endpoints: ["client1:eth1", "server:eth1"]
    - endpoints: ["client2:eth1", "server:eth2"]
```

### 10.8 تشغيل المثال

```bash
sudo containerlab deploy -t lab-message.clab.yml
docker ps
docker logs -f clab-msg-lab-client2
```

من Terminal آخر أرسل رسالة من client1 إلى client2:

```bash
docker exec -it clab-msg-lab-client1 python /app/send_message.py 10.0.1.100 client1 client2 "Hello client2 from client1"
```

المتوقع في client1:

```text
DELIVERED to client2
```

والمتوقع في logs الخاصة بـ client2:

```text
FROM client1: Hello client2 from client1
```

---

## 11. إيقاف اللابات والتنظيف

```bash
cd ~/network-project
sudo containerlab destroy -t lab.clab.yml --cleanup

cd ~/network-project/message-demo
sudo containerlab destroy -t lab-message.clab.yml --cleanup

docker ps
```

---

## 12. مشاكل شائعة وحلول سريعة

| المشكلة | الحل |
|---|---|
| `docker: permission denied` | نفذ `sudo usermod -aG docker $USER` ثم افتح Ubuntu من جديد |
| `Cannot connect to Docker daemon` | شغل Docker باستخدام `sudo service docker start` |
| `containerlab: command not found` | أعد تثبيت Containerlab باستخدام أمر curl الرسمي |
| `No topology files found` | تأكد أنك داخل مجلد ملف `lab.clab.yml` أو استخدم `-t` مع المسار الصحيح |
| `ping` لا يعمل | افحص `ip addr` داخل كل node وتأكد من `links` في YAML |
| اسم الكونتينر غير معروف | استخدم `docker ps` لمعرفة الاسم الكامل |

---

## 13. Checklist قبل العرض

- [ ] Ubuntu يفتح بدون مشاكل.
- [ ] `docker run hello-world` يعمل.
- [ ] `containerlab version` يظهر رقم الإصدار.
- [ ] `deploy` يعمل بدون أخطاء.
- [ ] `docker ps` يظهر server وclients.
- [ ] ping من client1 إلى server ناجح.
- [ ] ping من client2 إلى server ناجح.
- [ ] مثال الرسائل يعمل: client1 يرسل وclient2 يستقبل عبر server.

---

## 14. مراجع رسمية مفيدة

- Microsoft WSL installation documentation: <https://learn.microsoft.com/windows/wsl/install>
- Ubuntu on WSL: <https://ubuntu.com/wsl>
- Docker Engine on Ubuntu documentation: <https://docs.docker.com/engine/install/ubuntu/>
- Containerlab installation documentation: <https://containerlab.dev/install/>
- Containerlab topology documentation: <https://containerlab.dev/manual/topo-def-file/>

---

**Prepared by: izzaldeen mansour**
