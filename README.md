
    Create users deploy_view and deploy_edit. Give the user deploy_view rights only to view deployments, pods. Give the user deploy_edit full rights to the objects deployments, pods.
    Create namespace prod. Create users prod_admin, prod_view. Give the user prod_admin admin rights on ns prod, give the user prod_view only view rights on namespace prod.
    Create a serviceAccount sa-namespace-admin. Grant full rights to namespace default. Create context, authorize using the created sa, check accesses.

openssl genrsa -out deploy_view.key 2048
openssl req -new -key deploy_view.key -out deploy_view.csr -subj "//CN=deploy_view"
openssl x509 -req -in deploy_view.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out deploy_view.crt -days 500
kubectl config set-credentials deploy_view --client-certificate=deploy_view.crt --client-key=deploy_view.key
kubectl config set-context deploy_view --cluster=minikube --user=deploy_view
kubectl apply -f binding.yaml
kubectl config use-context deploy_view
kubectl auth can-i create deployments --namespace kube-system

openssl genrsa -out deploy_edit.key 2048
openssl req -new -key deploy_edit.key -out deploy_edit.csr -subj "//CN=deploy_edit"
openssl x509 -req -in deploy_edit.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out deploy_edit.crt -days 500
kubectl config set-credentials deploy_edit --client-certificate=deploy_edit.crt --client-key=deploy_edit.key
kubectl config set-context deploy_edit --cluster=minikube --user=deploy_edit
kubectl apply -f binding.yaml
kubectl config use-context deploy_edit
kubectl auth can-i create deployments --namespace kube-system

openssl genrsa -out prod_view.key 2048
openssl req -new -key prod_view.key -out prod_view.csr -subj "//CN=prod_view"
openssl x509 -req -in prod_view.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out prod_view.crt -days 500
kubectl config set-credentials prod_view --client-certificate=prod_view.crt --client-key=prod_view.key
kubectl config set-context prod_view --cluster=minikube --user=prod_view
kubectl apply -f ./prod_view/binding.yaml
kubectl config use-context prod_view
kubectl auth can-i create deployments --namespace prod

openssl genrsa -out prod_admin.key 2048
openssl req -new -key prod_admin.key -out prod_admin.csr -subj "//CN=prod_admin"
openssl x509 -req -in prod_admin.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out prod_admin.crt -days 500
kubectl config set-credentials prod_admin --client-certificate=prod_admin.crt --client-key=prod_admin.key
kubectl config set-context prod_admin --cluster=minikube --user=prod_admin
kubectl apply -f ./prod_admin/binding.yaml
kubectl config use-context prod_admin
kubectl auth can-i create deployments --namespace prod

kubectl apply -f sa.yaml
kubectl describe sa sa-default-admin
kubectl describe secret <token_name>
kubectl config set-credentials sa-default-admin --token=<token>
kubectl config set-context sa-default-context --cluster=minikube --user=sa-default-admin
kubectl apply -f binding.yaml
kubectl config use-context sa-default-context
kubectl auth can-i create deployments --namespace default