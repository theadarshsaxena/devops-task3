pipeline{
  agent any

job("JOB1a"){
  description("Automatically download the code from github. Create Dynamic image from dockerfile and upload it to docker hub.")
  scm{
    github("theadarshsaxena/devops-task3", 'master')
  }
  triggers{
    scm("* * * * * ")
  }
  steps{
    shell('''sudo cp -r -v -f * /jendata''')
  }
}

job("JOB2a"){
  description("For deploying in k8s")
  triggers{
    upstream {
      upstreamProjects("JOB1a")
      threshold("SUCCESS")
    }
  }
  steps{
    shell('''if sudo kubectl get pvc | grep my-httpd-pvc
then
	echo "PVC already present"
	echo "changing PVC"
	sudo kubectl apply -f  /jendata/mypvc.yml
else
	echo "Creating PVC"
	sudo kubectl create -f /jendata/mypvc.yml
fi

if sudo kubectl get svc | grep myserver
then
	echo "Already present, hence changing conf"
	sudo kubectl apply -f /jendata/svc.yml
fi

if sudo kubectl get deployment | grep mywebdeploy
then
	echo "Deployment already present, hence, rolling out"
	sudo kubectl rollout restart deployment/mywebdeploy
else
	echo "creating deployment"
	sudo kubectl create -f /jendata/Deployment.yml
fi''')
  }
}


job("JOB3a"){
  description("Testing the Code.")
  triggers{
    upstream {
      upstreamProjects("JOB2a")
      threshold("SUCCESS")
    }
  }
  steps{
    shell('''sudo sleep 40
status=$(curl -o /dev/null -s -w "%{http_code}" http://192.168.99.103:30251/index.html)
if [ $status == 200 ]
then
    echo "running fine"
    exit 0
else
    echo "error with Status_code : $status"
    exit 1
fi''')
  }
}
}
