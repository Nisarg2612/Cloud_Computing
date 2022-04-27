# Automated Essay Scoring System with Kubernetes 

The AES system utilizes the **Deep Learning BERT** (Bidirectional Encoder Representations from Transformers) model to perform **text summarization** of Essay. The model was trained with **BertSum** on **Ubuntu 20.04 Virtual OS** with **300K files** of the **CNN stories**. Once the summary is completed, the **supervised leaning of machine learning** method, compares the instructor’s summary with the students’ one by using the **Naïve Bayes classification**.

<details>
<summary>To train and use the BERT model generate by BertSum</summary>
<a href=https://github.com/Quan25/flask-summary/blob/master/guides.pdf> Document</a>
</details>

## Run the Service on VM

Create a **new project** on **GCP**
![image](https://user-images.githubusercontent.com/35762778/165622491-e245908d-93e1-443e-aeeb-734cd8192cde.png)

Select the project that you created and **open the GCP terminal windows**

![image](https://user-images.githubusercontent.com/35762778/165637720-54f23c0f-75cd-4765-a07e-5c6008b1f4d0.png)

Launch the new aes-project **Kubernetes cluster** on GKE

```bash
gcloud container clusters create aes-project --num-nodes=1 --machine-type=e2-micro --region=us-west1
```
![image](https://user-images.githubusercontent.com/35762778/165637962-5cbfca3c-993a-40a6-90ed-62787e70333d.png)

```bash
kubectl get nodes
```
![image](https://user-images.githubusercontent.com/35762778/165640015-7ae5c1d6-fcdb-4767-a729-2c908462bbe6.png)


Make new **final_project** directory and go to the created final_project directory.
```bash
mkdir final_project
cd final_project
```

**Download** the latest Anaconda **Miniconda shell script**
```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

**Execute** and **run the miniconda** installation script
```bash
chmod +x Miniconda3-latest-Linux-x86_64.sh
./Miniconda3-latest-Linux-x86_64.sh
```

Create and **activate a Python virtual environment** on GCP cloud shell
```bash
conda create -n myenv python=3.6
conda activate myenv
```
![image](https://user-images.githubusercontent.com/35762778/165640059-cccdcefa-741b-456d-a7a1-4ce8f856959a.png)

**Clone the Quan’s AES project** from github.
```bash
git clone https://github.com/Quan25/flask-summary.git aes_project
cd aes_project
```
![image](https://user-images.githubusercontent.com/35762778/165639949-1e4386d6-d805-49b7-a6cc-369982dff750.png)

**Download and unzip** the rouge package
```bash
wget --load-cookies /tmp/cookies.txt "https://docs.google.com/uc?export=download&confirm=$(wget --quiet --savecookies /tmp/cookies.txt --keep-session-cookies --no-check-certificate 'https://drive.google.com/file/d/1RxfZOYyNvzvCf37_vABfJMkohAsEZKtH/' -O- | sed -rn 's/.confirm=([0-9A-Za-z_]+)./\1\n/p')&id=1RxfZOYyNvzvCf37_vABfJMkohAsEZKtH" -O rough.zip && rm -rf /tmp/cookies.txt
unzip rough.zip
```
Install and test **rouge** package
```bash
cd RELEASE-1.5.5
sudo apt-get install libxml-parser-perl
sudo cpan install XML::Parser::PerlSAX
sudo cpan install XML::RegExp
sudo cpan install XML::DOM
./runROUGE-test.pl
```
![image](https://user-images.githubusercontent.com/35762778/165639843-824ce61c-255b-4b64-8d28-d516a8a9ff78.png)

Install **pyrouge python wrapper**
```bash
git clone https://github.com/bheinzerling/pyrouge.git
cd pyrouge
pip install -e .
```
Download **pretrained-bert-model**
```bash
cd ..
cd ..
wget https://s3.amazonaws.com/models.huggingface.co/bert/bert-large-uncased.tar.gz
```
![image](https://user-images.githubusercontent.com/35762778/165640192-1a72e819-0908-4c1f-9b44-addef1eb1f0f.png)

**Change the path** in **BertParent.py**
```bash
cd summarizer
# from
self.model = BertModel.from_pretrained('/home/quan/Downloads/bert-large-uncased')
# to
self.model = BertModel.from_pretrained('/home/your-username/bert-large-uncased.tar.gz')
```
![image](https://user-images.githubusercontent.com/35762778/165640227-10416638-2d06-4274-8662-dbc7294ed597.png)

**Install flask** and related python **libraries**
```bash
pip3 install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cpu
pip3 install flask pandas sklearn nltk
pip3 install gensim==3.8.3
pip3 install pytorch-pretrained-bert
pip3 install matplotlib==3.0.1
```
Download **punkt package** with nltk
```bash
python3
import nltk
nltk.download(‘punkt’)

```
## Run the AES flask app 
```bash
python3 app.py
```
![image](https://user-images.githubusercontent.com/35762778/165640267-ab0896d4-36f3-4c2f-8140-05917372ac97.png)

Go to web preview and **chage the port to 5000**

![image](https://user-images.githubusercontent.com/35762778/165640290-74c49693-48b1-4508-b081-53fb4344fc63.png)
![image](https://user-images.githubusercontent.com/35762778/165640315-12b891fe-ef8b-4a35-9214-a1b84a4077b9.png)

Make the **text summary**

![image](https://user-images.githubusercontent.com/35762778/165640786-eee4491c-1a63-4b65-8429-94756efd8761.png)


## Deploy the system on GCP with docker container

Create the docker hub aes repository for this automated essay scoring system here

![image](https://user-images.githubusercontent.com/35762778/165640969-e38f7aca-1a04-4d4b-9131-72a1012c3618.png)

Create Dockerfile 
```bash
FROM python:alpine3.7
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt 
ENV PORT 5000
EXPOSE 5000
ENTRYPOINT [ "python3" ]
CMD [ "app.py" ]
```

Build the docker image.
```bash
docker build -t yourdockerhubID/aes .
```
Push the aes system docker image to docker hub 
```bash
docker push yourdockerhubID/aes
```

## To test the image on the GCP cloudshell

Run the docker image
```bash
docker run -p 5000:5000 -t yourdockerhubID /aes
```

## Host the system on GCP for external users
Start minikube
```bash
minikube start
```
![image](https://user-images.githubusercontent.com/35762778/165641474-6cb24e34-6bb2-4f01-865d-0f4bd96b2fbb.png)

Enable Ingress service on minikube for domain name
```bash
minikube addons enable ingress
```
Create depoyment file 
```bash
vim aes-project-deployment.yaml
```
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aes-deployment
spec:
  selector:
    matchLabels:
      app: aes-deployment
  replicas: 1
  template:
    metadata:
      labels:
        app: aes-deployment
    spec:
      containers:
      - name: aes-deployment
        image: your-dockerhub-id/aes
        ports:
        - containerPort: 5000
```
Launch the deployment with aes-project-deployment.yaml
```bash
kubectl apply -f aes-deployment.yaml
```
![image](https://user-images.githubusercontent.com/35762778/165642140-c16e844f-81c8-4546-b516-515fa459874a.png)

Create a service file to access the system pod
```bash
vim aes-project-service.yaml
```
```bash
apiVersion: v1
kind: Service
metadata:
  name: aes-service
spec:
  selector:
    app: aes-deployment
  ports:
  - protocol: TCP
    port: 5000
    targetPort: 5000
```
Launch the service with using the aes-project-service.yaml
```bash
kubectl apply -f aes-service.yaml
```
![image](https://user-images.githubusercontent.com/35762778/165642156-09d0de3a-0d25-4538-96a7-456652223279.png)

Create an aes-ingress.yaml file for domain name 
```bash
vim aes-ingress.yaml
```
```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: aes-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
    - host: cs571.aesproject.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: aes-service
                port:
                  number: 5000
```

Launch the ingress service
```bash
kubectl apply -f aes-ingress.yaml
```
![image](https://user-images.githubusercontent.com/35762778/165642176-4a35c074-feb5-4a4d-a7c3-fb949e127e67.png)


Get the **IP-Address of the nginx server**
```bash
kubectl get ingress
```
![image](https://user-images.githubusercontent.com/35762778/165641668-7b8550d1-8b01-419e-b0a0-1cb4bb85caac.png)

Add IP-ADDRESS to /etc/hosts
```bash
sudo vi /etc/hosts
```
**Add the address** you got from above step to the **end of the file**

![image](https://user-images.githubusercontent.com/35762778/165641726-0d912574-3af9-4fa7-b579-369e72f345e1.png)

Check the full **cluster configuration state**
```bash
kubectl get all 
```
![image](https://user-images.githubusercontent.com/35762778/165641768-68b52cb4-84ff-4822-959d-d57894ea4cb1.png)

**Access the application** from **search browser** with domain name 
```bash
cs571.aesproject.com
```
![image](https://user-images.githubusercontent.com/35762778/165641812-0275db6b-bf7c-4d22-8b81-590cd622ff31.png)

To get the **student score** click on the **Grade Students** button and you will **get the result**

![image](https://user-images.githubusercontent.com/35762778/165641833-ab40ac75-e2b8-4dc5-8d29-df0b1175c6fe.png)
