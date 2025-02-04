# deploy.sh
nano deploy.sh
#!/bin/bash

SERVER_IP="65.21.158.68"
SERVER_USER="root"
APP_NAME="my-app"

echo "Building Docker Image..."
docker build -t $APP_NAME:latest .

echo "Saving Docker Image..."
docker save $APP_NAME:latest | gzip > $APP_NAME.tar.gz

echo "Transferring Docker Image to Server..."
scp -o StrictHostKeyChecking=no $APP_NAME.tar.gz $SERVER_USER@$SERVER_IP:/root/

echo "Deploying on HUC Cloud Server..."
ssh -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP << EOF
  docker load < /root/$APP_NAME.tar.gz
  docker stop $APP_NAME || true
  docker rm $APP_NAME || true
  docker run -d --name $APP_NAME -p 80:80 $APP_NAME:latest
EOF

echo "Deployment Completed!"
chmod +x deploy.sh./deploy.sh

