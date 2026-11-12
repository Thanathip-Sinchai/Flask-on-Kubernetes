# รันเว็บ Flask บน Kubernetes

ทำตามทีละขั้นตามลำดับ .

สิ่งที่จะได้เมื่อทำเสร็จ: เปิด http://localhost:3000 แล้วเห็นข้อความ Hello World Gus โดยเว็บนี้รันอยู่บน Kubernetes 2 ชุด

ใช้เครื่องมือเหล่านี้: Windows, VS Code, PowerShell (อยู่ใน VS Code), Git, Docker Desktop

---

## สารบัญ

0. [เตรียมเครื่อง](#ขั้นที่-0-เตรียมเครื่อง)
1. [ดาวน์โหลดโปรเจกต์ด้วย git clone](#ขั้นที่-1-ดาวน์โหลดโปรเจกต์ด้วย-git-clone)
2. [เปิด Terminal](#ขั้นที่-2-เปิด-terminal)
3. [ดูไฟล์ในโปรเจกต์](#ขั้นที่-3-ดูไฟล์ในโปรเจกต์)
4. [ตรวจว่า Docker และ Kubernetes พร้อม](#ขั้นที่-4-ตรวจว่า-docker-และ-kubernetes-พร้อม)
5. [สร้าง Docker image](#ขั้นที่-5-สร้าง-docker-image)
6. [สั่ง Kubernetes ให้รันแอป](#ขั้นที่-6-สั่ง-kubernetes-ให้รันแอป)
7. [ตรวจว่าแอปทำงาน](#ขั้นที่-7-ตรวจว่าแอปทำงาน)
8. [เปิดเว็บ](#ขั้นที่-8-เปิดเว็บ)
9. [งานที่ทำต่อได้](#งานที่ทำต่อได้) (เปิดใหม่ / แก้โค้ด / ลบทิ้ง / ลองรันบนเครื่อง)

---

## ภาพรวมแบบสั้น ๆ

```text
เขียนเว็บ (app.py)
      ↓  Dockerfile แพ็กเป็น "กล่อง"
Docker image (กล่องสำเร็จรูป)
      ↓  deployment.yaml สั่ง Kubernetes
Kubernetes รันกล่อง 2 ชุด + มีทางเข้า (Service)
      ↓  kubectl port-forward
เปิดเว็บที่ http://localhost:3000
```

### คำศัพท์ที่จะเจอ

| คำ | ความหมายแบบง่าย |
|---|---|
| Flask | เครื่องมือเขียนเว็บด้วย Python |
| Docker image | กล่องสำเร็จรูปที่รวมเว็บกับของที่ต้องใช้ พร้อมรันได้ทุกเครื่อง |
| Kubernetes | ระบบที่คอยรันและดูแลกล่องให้ทำงานอยู่เสมอ |
| Pod | หนึ่งชุดของแอปที่ Kubernetes รัน (งานนี้มี 2 ชุด) |
| Deployment | ใบสั่งงานบอก Kubernetes ว่าจะให้รันกี่ชุด |
| Service | ทางเข้ากลางที่ส่งคำขอไปยัง Pod |
| Terminal / PowerShell | หน้าต่างที่เราพิมพ์คำสั่ง |
| Git / GitHub | Git คือเครื่องมือเก็บไฟล์โปรเจกต์ GitHub คือเว็บที่เก็บโปรเจกต์ไว้ให้ดาวน์โหลด |
| git clone | คำสั่งดาวน์โหลดโปรเจกต์ทั้งก้อนจาก GitHub ลงเครื่องเรา |

---

## ขั้นที่ 0: เตรียมเครื่อง

ติดตั้งให้ครบก่อน (ถ้ามีแล้วข้ามได้)

- [ ] Python
- [ ] VS Code
- [ ] Git (ใช้ดาวน์โหลดโปรเจกต์ ติดตั้ง Git for Windows จาก https://git-scm.com)
- [ ] Docker Desktop (มี kubectl มาให้ในตัวโดยทั่วไป)

จากนั้นเปิด Docker Desktop แล้วเปิด Kubernetes ในหน้า Settings ของโปรแกรม รอจนสถานะ Kubernetes พร้อมใช้งาน

---

## ขั้นที่ 1: ดาวน์โหลดโปรเจกต์ด้วย git clone

git clone คือคำสั่งที่ดาวน์โหลดโปรเจกต์ทั้งก้อนจาก GitHub ลงเครื่องเรา ได้ไฟล์ครบทุกไฟล์พร้อมใช้ ไม่ต้องสร้างและวางโค้ดเอง

1. เปิด VS Code แล้วเลือก Terminal > New Terminal ตรวจว่าเป็น PowerShell
2. สร้างโฟลเดอร์ code สำหรับเก็บงาน แล้วเข้าไปในโฟลเดอร์นั้น (ถ้ามีโฟลเดอร์อยู่แล้วก็ไม่เป็นไร):

```powershell
mkdir -Force C:\code
cd C:\code
```

3. ดาวน์โหลดโปรเจกต์:

```powershell
git clone https://github.com/Thanathip-Sinchai/Flask-on-Kubernetes.git
```

ถ้าสำเร็จจะได้โฟลเดอร์ Flask-on-Kubernetes อยู่ใน C:\code

4. ใน VS Code เลือก File > Open Folder... เลือกโฟลเดอร์ C:\code\Flask-on-Kubernetes แล้วกด Select Folder (ถ้ามีหน้าต่างถามว่าเชื่อถือโฟลเดอร์นี้หรือไม่ ให้เลือก Yes, I trust the authors)

> ถ้าขึ้นว่า git is not recognized แปลว่ายังไม่ได้ติดตั้ง Git ให้ติดตั้งแล้วปิดเปิด VS Code ใหม่
> ถ้าเป็น repository แบบ private ระบบจะให้ล็อกอิน GitHub ในครั้งแรก

---

## ขั้นที่ 2: เปิด Terminal

1. ใน VS Code เลือก Terminal > New Terminal (ตอนนี้เปิดโปรเจกต์อยู่ Terminal จะเริ่มในโฟลเดอร์โปรเจกต์)
2. จะมีหน้าต่างขึ้นด้านล่าง ให้ตรวจว่าเป็น PowerShell (ถ้าไม่ใช่ ให้เลือก PowerShell)
3. ดูบรรทัดที่รอให้พิมพ์ ควรเห็นชื่อโฟลเดอร์โปรเจกต์ แบบนี้:

```text
PS C:\code\Flask-on-Kubernetes>
```

ถ้าเห็นโฟลเดอร์อื่น ให้พิมพ์:

```powershell
cd C:\code\Flask-on-Kubernetes
```

> กฎสำคัญ: คำสั่งในกล่อง powershell ทุกอัน ให้พิมพ์ (หรือวาง) ใน Terminal นี้ แล้วกด Enter
> ส่วนเนื้อหาในกล่องอื่น (python, yaml ฯลฯ) เป็นเนื้อหาของไฟล์ ไม่ต้องพิมพ์ใน Terminal

---

## ขั้นที่ 3: ดูไฟล์ในโปรเจกต์

ไฟล์ทั้งหมดมากับโปรเจกต์แล้ว ไม่ต้องสร้างเอง ดูที่แถบ Explorer ด้านซ้ายของ VS Code ควรเห็นไฟล์เหล่านี้:

```text
app.py
requirements.txt
.flaskenv
Dockerfile
deployment.yaml
README.md
```

ส่วนนี้ไว้อ่านทำความเข้าใจว่าแต่ละไฟล์ทำอะไร ถ้าไม่สนใจข้ามไปขั้นที่ 4 ได้เลย

### ไฟล์ที่ 1: app.py (ตัวเว็บ)

```python
from flask import Flask

app = Flask(__name__)


@app.get("/")
def hello():
    return "Hello World Gus"


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=3000)
```

อธิบาย: เมื่อมีคนเปิดหน้าแรกของเว็บ (/) โปรแกรมจะตอบกลับด้วยข้อความ Hello World Gus

### ไฟล์ที่ 2: requirements.txt (รายการแพ็กเกจที่ต้องใช้)

```text
Flask==3.1.2
python-dotenv==1.1.1
gunicorn==23.0.0; sys_platform != "win32"
```

อธิบาย: บอกว่าโปรเจกต์ต้องใช้ Flask (สร้างเว็บ), python-dotenv (อ่านไฟล์ตั้งค่า) และ Gunicorn (ตัวรันเว็บใน Linux container) บรรทัด Gunicorn มีเงื่อนไขให้ข้ามบน Windows เพราะไม่ได้ใช้บน Windows

### ไฟล์ที่ 3: .flaskenv (ตั้งค่าตอนลองรันบนเครื่อง)

```env
FLASK_APP=app
FLASK_RUN_HOST=0.0.0.0
FLASK_RUN_PORT=3000
```

อธิบาย: ใช้เฉพาะตอนลองรันเว็บบนเครื่องตรง ๆ (ส่วนทางเลือกท้ายคู่มือ) ถ้าไม่ลองก็ไม่ได้ใช้

### ไฟล์ที่ 4: Dockerfile (สูตรทำกล่อง)

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
EXPOSE 3000
CMD ["gunicorn", "--bind", "0.0.0.0:3000", "app:app"]
```

อธิบายทีละบรรทัด:

| บรรทัด | ทำอะไร |
|---|---|
| FROM python:3.12-slim | เริ่มจากกล่องที่มี Python อยู่แล้ว |
| WORKDIR /app | ใช้โฟลเดอร์ /app ในกล่องเป็นที่ทำงาน |
| COPY requirements.txt . | ก๊อปรายการแพ็กเกจเข้ากล่อง |
| RUN pip install ... | ติดตั้งแพ็กเกจ |
| COPY app.py . | ก๊อปตัวเว็บเข้ากล่อง |
| EXPOSE 3000 | ระบุว่าเว็บใช้พอร์ต 3000 |
| CMD [...] | คำสั่งเริ่มเว็บด้วย Gunicorn (app:app = ไฟล์ app.py + ตัวแปรชื่อ app) |

### ไฟล์ที่ 5: deployment.yaml (ใบสั่งงาน Kubernetes)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-demo
  template:
    metadata:
      labels:
        app: flask-demo
    spec:
      containers:
        - name: flask-demo
          image: flask-k8s:1.0
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 3000
          readinessProbe:
            httpGet:
              path: /
              port: http
            initialDelaySeconds: 3
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /
              port: http
            initialDelaySeconds: 10
            periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: flask-demo
spec:
  type: ClusterIP
  selector:
    app: flask-demo
  ports:
    - name: http
      port: 3000
      targetPort: http
```

> ไฟล์ YAML ต้องเว้นวรรค (indent) ให้ตรงตามตัวอย่าง ถ้าแก้ไฟล์ให้ระวังอย่าลบหรือเพิ่มช่องว่าง และบรรทัด --- คือเส้นคั่นระหว่าง 2 ส่วน

อธิบาย: ไฟล์นี้มี 2 ส่วน

- Deployment — ให้รันแอป 2 ชุด (replicas: 2) จาก image flask-k8s:1.0
  - readinessProbe ตรวจว่าแอปพร้อมรับคำขอหรือยัง
  - livenessProbe ตรวจว่าแอปยังไม่ค้าง
- Service — ทางเข้ากลางที่ส่งคำขอไปยัง Pod

✅ เช็กก่อนไปต่อ: ใน Explorer ต้องเห็นไฟล์ครบตามรายการข้างบน

---

## ขั้นที่ 4: ตรวจว่า Docker และ Kubernetes พร้อม

4.1 ตรวจ Docker

```powershell
docker --version
```

✅ ควรเห็นเลขเวอร์ชัน
❌ ถ้าขึ้นว่าไม่พบคำสั่ง ให้ติดตั้งหรือเปิด Docker Desktop ก่อน

4.2 ตรวจ Kubernetes

```powershell
kubectl config current-context
kubectl get nodes
```

✅ ควรเห็น docker-desktop และ node ที่สถานะ Ready
❌ ถ้าเชื่อมต่อไม่ได้ ให้เปิด Docker Desktop รอให้ Kubernetes พร้อม แล้วลองใหม่

---

## ขั้นที่ 5: สร้าง Docker image

```powershell
docker build -t flask-k8s:1.0 .
```

อธิบาย: สั่งให้ Docker อ่าน Dockerfile แล้วสร้างกล่องชื่อ flask-k8s เวอร์ชัน (tag) 1.0 — อย่าลืมจุด . ท้ายคำสั่ง (แปลว่าใช้ไฟล์ในโฟลเดอร์ปัจจุบัน)

✅ รอจนจบโดยไม่มี error จากนั้นตรวจ:

```powershell
docker image ls flask-k8s
```

ควรเห็น flask-k8s และ tag 1.0 ในรายการ

> Docker Desktop สร้าง image และรัน Kubernetes บนเครื่องเดียวกัน จึงใช้ image นี้ได้ทันที
> (ถ้าใช้ Kubernetes บนเครื่องอื่น ต้องอัปโหลด image ขึ้น container registry แล้วแก้ image: ใน deployment.yaml)

---

## ขั้นที่ 6: สั่ง Kubernetes ให้รันแอป

```powershell
kubectl apply -f deployment.yaml
```

อธิบาย: ส่ง deployment.yaml ให้ Kubernetes แล้วให้ทำตามนั้น

✅ ครั้งแรกควรเห็น:

```text
deployment.apps/flask-demo created
service/flask-demo created
```

> ถ้าเห็น unchanged หรือ configured ไม่ใช่ข้อผิดพลาด แปลว่าเคยสร้างไว้แล้วหรือมีการอัปเดต

---

## ขั้นที่ 7: ตรวจว่าแอปทำงาน

7.1 รอให้แอปเริ่ม

```powershell
kubectl rollout status deployment/flask-demo
```

✅ ควรเห็น deployment "flask-demo" successfully rolled out

7.2 ดู Pod

```powershell
kubectl get pods
```

✅ ควรเห็น 2 รายการที่ขึ้นต้นด้วย flask-demo- โดยแต่ละรายการ READY = 1/1 และ STATUS = Running

7.3 ดู Service

```powershell
kubectl get services
```

✅ ควรเห็น flask-demo และพอร์ต 3000/TCP

ถ้าผ่านทั้ง 3 ข้อ แปลว่าแอปพร้อมทำงานใน Kubernetes แล้ว

---

## ขั้นที่ 8: เปิดเว็บ

Service ชนิดนี้เข้าได้เฉพาะภายใน Kubernetes จึงต้องเชื่อมพอร์ตมาที่เครื่องเรา:

```powershell
kubectl port-forward service/flask-demo 3000:3000
```

✅ ควรเห็น:

```text
Forwarding from 127.0.0.1:3000 -> 3000
```

> ⚠️ ปล่อย Terminal นี้เปิดค้างไว้ (กด Ctrl+C = ปิดการเชื่อมต่อ)
> ถ้าต้องพิมพ์คำสั่งอื่นต่อ ให้เปิด Terminal ใหม่ด้วยปุ่ม + ที่แถบ Terminal

จากนั้นเปิดเบราว์เซอร์ไปที่:

```text
http://localhost:3000
```

🎉 ควรเห็นข้อความ Hello World Gus — เสร็จแล้ว!

---

## งานที่ทำต่อได้

### ปิดแล้วเปิดเครื่องใหม่ (เปิดเว็บอีกครั้ง)

1. เปิด Docker Desktop รอให้ Kubernetes พร้อม
2. เปิดโฟลเดอร์โปรเจกต์ใน VS Code แล้วเลือก Terminal > New Terminal
3. ตรวจด้วย kubectl get pods และ kubectl get services
4. ถ้าไม่มี Deployment/Service ให้รัน kubectl apply -f deployment.yaml
5. รัน kubectl port-forward service/flask-demo 3000:3000 แล้วเปิด http://localhost:3000

ถ้าไม่ได้แก้โค้ด ไม่ต้อง build image ใหม่

### แก้โค้ดแล้วอยากอัปเดตเว็บ

1. แก้ app.py แล้วกด Ctrl+S
2. สร้าง image เวอร์ชันใหม่ (เปลี่ยนเลข tag):

   ```powershell
   docker build -t flask-k8s:1.1 .
   ```

3. เปิด deployment.yaml แก้บรรทัดนี้ แล้วกด Ctrl+S:

   ```yaml
   image: flask-k8s:1.0
   ```

   ให้เป็น:

   ```yaml
   image: flask-k8s:1.1
   ```

4. สั่งอัปเดต:

   ```powershell
   kubectl apply -f deployment.yaml
   kubectl rollout status deployment/flask-demo
   kubectl get pods
   ```

5. เมื่อ Pod ใหม่เป็น Running และ 1/1 ให้รีเฟรชหน้าเว็บ (ถ้า port-forward หยุดไป ให้รันใหม่)

### ลบแอปออกจาก Kubernetes

```powershell
kubectl delete -f deployment.yaml
```

### (ทางเลือก) ลองรันเว็บบนเครื่องก่อนใช้ Kubernetes

ขั้นนี้ไม่จำเป็น ข้ามได้เลย ใช้เมื่ออยากลองเว็บแบบไม่ผ่าน Docker

```powershell
py -m venv .venv
Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python -m flask run
```

- ถ้าสำเร็จจะเห็น (.venv) หน้าบรรทัดคำสั่ง
- เปิด http://localhost:3000 ควรเห็น Hello World Gus
- กด Ctrl+C เพื่อหยุดเว็บก่อนทำขั้นอื่นต่อ (พอร์ต 3000 จะได้ว่าง)

---

## สรุป

| ขั้น | ทำอะไร | คำสั่งหลัก |
|---|---|---|
| 1–3 | ดาวน์โหลดโปรเจกต์ เปิดใน VS Code และดูไฟล์ | git clone ... |
| 4 | ตรวจเครื่องมือ | docker --version, kubectl get nodes |
| 5 | สร้างกล่อง (image) | docker build -t flask-k8s:1.0 . |
| 6 | สั่ง Kubernetes รันแอป | kubectl apply -f deployment.yaml |
| 7 | ตรวจว่าทำงาน | kubectl rollout status deployment/flask-demo, kubectl get pods |
| 8 | เปิดเว็บ | kubectl port-forward service/flask-demo 3000:3000 |

งานนี้ไม่ใช้ Docker Compose (compose.yaml) เพราะสั่ง Kubernetes โดยตรงผ่าน deployment.yaml
