# Kubernetes - Grupo3
Wordpress - MySQL

# Atividade Kubetnetes

## Objetivo:
- Aplicar os conhecimentos adquiritos na trilha de Kubernetes

## Instruções:

1º - Crie um namespace chamado labwordpress, tudo o que for feito deve estar dentro deste namespace;	

2º - Faça apply do arquivo de service do MYSQL  mude a porta padrão para 3308;

3º - Crie o arquivo secret que deverá conter o password do banco MYSQL , lembre-se de criar uma senha forte;

4º - Faça o apply do arquivo de PersintentVolumeClaim do MYSQL para uma capacidade de 3GB;

5º - Faça o Apply do arquivo de Deplyoment do MYSQL chamado "mysql-persintent-storage-lab", apontando para o /var/lib/mysql. Lembre-se de criar o volume em si com o mesmo nome do volume mount;

6º - Faça o apply do arquivo de service do Wordpress altere para tcp PORT 80;

7º - Faça o apply do arquivo de PersintentVolumeClaim do Wordpress, para uma capacity de 3GB;

8º - Faça o Apply do arquivo de Deplyoment do Wordpress chamado "wordpress-persintent-storage-lab", apontando para o /var/www/html. Lembre-se de criar o volumeem si com o mesmo nome do volume mount;

9º - No arquivo de deployment do wordpress, insira o secret contendo o password do MYSQL, criado no começo do exercícios;

10º - Faça o apply do arquivo de deployment do Wordpress;

11º - Verifique se os Pods, Service e os PVC's foram criados de forma corretas dentro do namespace criado no inicio deste exercício;

12º - Verifique qual a URL gerada através do ingress do kubernetes;

13º - Copie essa URL do ingress e cole no Browser para abrir a tela inicial do wordpress;

14º - Criar documentação.

# Realizando as Intruções:

## Criando um namespace chamado labwordpress:

1º Utilizamos o comando `kubectl create namespace labwordpress`;

2º Se for para configurar que todos os comandos kubectl salve pernametemente no namespace pode utlizar o comando `kubectl config set-context --current --namespace=labwordpress`
	
## Fazendo o apply do arquivo de service do MYSQL na porta 3308:
	
1º Iremos criar um arquivo no VScode e iremos criar um códimo em YAML:
	

~~~	apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  namespace: labwordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 3308
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
~~~	

2º	Salve o arquivo na pasta labwordpress criado no windows e execute no Powershell o comando `kubectl apply -f ./svc-mysql.yaml`

## Criando o arquivo secret do banco MYSQL:

1º Iremos criar o arquivo mysql-password:

~~~ apiVersion: v1
kind: Secret
metadata:
  name: mysecret
  namespace: labwordpress
type: Opaque
data:
  password: 3gRuPOdKPy
 ~~~
 
 2º Executar o arquivo no PowerShell do Windows:
 
 ` kubectl apply -f .\mysql-password.yaml `
 
 ## Realizando o apply do arquivo de PersintentVolumeClaim do MYSQL com Capacity de 3GB:
 
 1º Iremos criar o arquivo pvc-mysql:
 
~~~ apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  namespace: labwordpress
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
~~~

2º Executando a Apply no Powershell do Windows:

` kubectl apply -f .\pvc-mysql.yaml `


## Realizando o apply do arquivo de deployment do MYSQL apontaod para o /var/lib/mysql:

1º Iremos criar o arquivo dpt-mysql:

~~~ 
apiVersion: v1
kind: Deployment
metadata:
  name: wordpress-mysql
  namespace: labwordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
~~~ 

2º Executando o arquivo no Powershell:

` kubectl apply -f .\dpt-mysql.yaml `

## Realizando o apply do arquivo de serviço do Wordpress na TCP port 80:

1º Iremos criar um arquivo svc-wordpress:

~~~ apiVersion: v1
kind: Service
metadata:
  name: svc-blog-grupo3
  namespace: labwordpress
  labels:
  app: wordpress
spec:
  type: NodePort
  ports:
    - port: 80
        protocol: TCP
  selector:
    app: blog-grupo3
    tier: frontend
~~~

2º Executando o arquivo no Powershell:

` kubectl apply -f .\svs-wordpress.yaml `

## Realizando o apply do arquivo de PersintentVolumeClaim do Wordpress com Capacity de 3GB:

1º Iremos criar o arquivo pvc-wordpress:
~~~ apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  namespace: labwordpress
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
~~~ 

## Criando o arquivo de deployment do Wordpress apontando para o /var/www/html, inserindo o secret do password do mysql no arquivo e dando apply no deployment:

~~~ apiVersion: v1
kind: Deployment
metadata:
  name: blog-grupo3
  namespace: labwordpress
  labels:
    app: blog-grupo3
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:6.0-apache
        name: wordpress
        env:
        - name: host-wp
          value: wordpress-mysql:3308
        - name: wp-passwd
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        ports:
        - containerPort: 80
          name: wordpress
        volumeMounts:
        - name: wordpresspersistent-storage-lab
          mountPath: \var\www\html
      volumes:
      - name: wordpresspersistent-storage-lab
        persistentVolumeClaim:
          claimName: wp-pv-claim
~~~
