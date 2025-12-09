# Les étapes pour installer zeppelin :

## Mise à jour du système : 
    sudo yum update -y
    sudo yum install -y wget git unzip vim net-tools
## Installer Java 8 ou 11 : (déjà installé pour l'utiliser en hadoop)
    sudo yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel
## Créer un utilisateur dédié pour Zeppelin : 
    sudo useradd zeppelin
    sudo passwd zeppelin
## Créer un dossier d’installation : (en root)
    sudo mkdir -p /opt/zeppelin
    sudo chown -R zeppelin:zeppelin /opt/zeppelin
## Télécharger Zeppelin :
    sudo su - zeppelin
    cd /opt/zeppelin
    wget https://downloads.apache.org/zeppelin/zeppelin-0.10.1/zeppelin-0.10.1-bin-all.tgz
## Décompresser :
    tar -xvzf zeppelin-0.10.1-bin-all.tgz
    mv zeppelin-0.10.1-bin-all zeppelin
## Créer les fichiers de configuration :
    cd /opt/zeppelin/zeppelin/conf
    cp zeppelin-env.sh.template zeppelin-env.sh
    cp zeppelin-site.xml.template zeppelin-site.xml
## Configurer Zeppelin pour Hadoop :
    nano zeppelin-env.sh
### Ajouter :
      export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.462.b08-4.el9.x86_64
      export HADOOP_HOME=/home/hadoop/hadoop
      export HADOOP_CONF_DIR=/home/hadoop/hadoop/etc/hadoop
## Donner accès à Zeppelin aux fichiers Hadoop : (en root)
      sudo usermod -aG hadoop zeppelin
      sudo chmod -R g+r /home/hadoop/hadoop/etc/hadoop

## Créer un dossier HDFS pour Zeppelin :
      su – hadoop
      start-dfs.sh
      start-yarn.sh
      hdfs dfs -mkdir -p /user/zeppelin
      hdfs dfs -chown zeppelin:zeppelin /user/zeppelin
## Démarrer Zeppelin :
      su - zeppelin
      cd /opt/zeppelin/zeppelin
      bin/zeppelin-daemon.sh start
## Vérifier :
      bin/zeppelin-daemon.sh status
## Accéder à Zeppelin :
      http://<IP_SERVEUR>:8080
## Si il y a un problème de l’affichage de site utilise :
      netstat -tulnp | grep 8080
### si tu trouves 127.0.0.1:8080 tu ne pourras pas y accéder depuis une autre machine.
### Donc modifier le fichier de configuration et ajoute : 
      nano /opt/zeppelin/zeppelin/conf/zeppelin-site.xml
      <?xml version="1.0" encoding="UTF-8"?>
      <configuration>
        <property>
          <name>zeppelin.server.addr</name>
          <value>0.0.0.0</value>
          <description>Listen on all network interfaces</description>
        </property>
        <property>
          <name>zeppelin.server.port</name>
          <value>8080</value>
          <description>Port for Zeppelin web UI</description>
        </property>
      </configuration>
## Redémarrer Zeppelin :
      cd /opt/zeppelin/zeppelin
      bin/zeppelin-daemon.sh restart
## Vérifier le port avec root :
      sudo netstat -tulnp | grep 8080
### Tu devrais voir quelque chose comme :
      tcp6       0      0 :::8080                  :::*     
### Si toujours inaccessible depuis une autre machine :
      sudo firewall-cmd --permanent --add-port=8080/tcp
      sudo firewall-cmd --reload
      sestatus
      sudo setenforce 0
### Redémarrer Zeppelin :
      sudo su - zeppelin
      cd /opt/zeppelin/zeppelin
      bin/zeppelin-daemon.sh restart
### Accéder à Zeppelin :
      http://<IP_SERVEUR>:8080

# La création du data set :

## Installation Python : 
	sudo yum install python3 -y

### Script pour creation du base de données :
  
  cat > generate_sales.py << 'EOF'
  import csv
  import random
  from datetime import datetime, timedelta
  products = {
      'Electronics': ['Laptop', 'Smartphone', 'Tablet', 'Headphones', 'Monitor', 'Keyboard', 'Mouse', 'Printer', 'Speaker', 'Camera'],
      'Clothing': ['T-Shirt', 'Jeans', 'Shoes', 'Jacket', 'Dress', 'Socks', 'Hat', 'Coat', 'Shirt', 'Skirt'],
      'Books': ['Novel', 'Textbook', 'Magazine', 'Comic', 'Dictionary'],
      'Home': ['Chair', 'Table', 'Lamp', 'Bed', 'Sofa']
  }
  regions = ['North', 'South', 'East', 'West']
  customers = [f'C{1000+i}' for i in range(50)]
  prices = {
      'Laptop': 999.99, 'Smartphone': 799.99, 'Tablet': 299.99,
      'Headphones': 89.99, 'Monitor': 199.99, 'Keyboard': 45.99,
      'Mouse': 25.50, 'Printer': 149.99, 'Speaker': 79.99,
      'Camera': 499.99, 'T-Shirt': 19.99, 'Jeans': 59.99,
      'Shoes': 79.99, 'Jacket': 129.99, 'Dress': 49.99,
      'Socks': 9.99, 'Hat': 24.99, 'Coat': 199.99,
      'Shirt': 34.99, 'Skirt': 39.99, 'Novel': 14.99,
      'Textbook': 89.99, 'Magazine': 5.99, 'Comic': 9.99,
      'Dictionary': 29.99, 'Chair': 89.99, 'Table': 199.99,
      'Lamp': 39.99, 'Bed': 499.99, 'Sofa': 899.99
  }
  with open('sales_large.csv', 'w', newline='') as csvfile:
      fieldnames = ['OrderID', 'Date', 'Product', 'Category', 'Price', 'Quantity', 'Region', 'CustomerID', 'Profit']
      writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
      writer.writeheader()
      
      start_date = datetime(2023, 1, 1)
      
      for i in range(500):
          # Choisir une catégorie aléatoire
          category = random.choice(list(products.keys()))
          product = random.choice(products[category])
          
          order_id = f"ORD{20000 + i}"
          date = (start_date + timedelta(days=random.randint(0, 364))).strftime('%Y-%m-%d')
          price = prices[product]
          quantity = random.randint(1, 5)
          region = random.choice(regions)
          customer_id = random.choice(customers)
          
          # Calculer le profit (20-40% du prix)
          profit_margin = random.uniform(0.2, 0.4)
          profit = round(price * quantity * profit_margin, 2)
          
          writer.writerow({
              'OrderID': order_id,
              'Date': date,
              'Product': product,
              'Category': category,
              'Price': price,
              'Quantity': quantity,
              'Region': region,
              'CustomerID': customer_id,
              'Profit': profit
          })
  
  print("✅ Dataset généré: sales_large.csv (500 lignes)")
  EOF
## Exécuter le script
python3 generate_sales.py

## Vérifier
head -5 sales_large.csv
wc -l sales_large.csv

## Créer u dossier hdfs au on va transfermer la data set
hdfs dfs -mkdir -p /user/zeppelin/sales_data
hdfs dfs -put sales_large.csv /user/zeppelin/sales_data/

# Vérifier l'upload
hdfs dfs -ls /user/zeppelin/sales_data/
hdfs dfs -cat /user/zeppelin/sales_data/sales_large.csv | head -3


