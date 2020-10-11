Цель: - уметь запускать HA и multi master PostgreSQL кластер в Kubernetes
Запустить HA и multi master PostgreSQL кластер в Kubernetes. Описать что и как делали и с какими проблемами столкнулись

1. Устанавливаем gke
2. Устанавливаем kubectl и google sdk (для винды 10)

	(New-Object Net.WebClient).DownloadFile("https://dl.google.com/dl/cloudsdk/channels/rapid/GoogleCloudSDKInstaller.exe", "$env:Temp\GoogleCloudSDKInstaller.exe")
	choco install kubernetes-cli
	gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project sodium-hangar-292213
	
3. Устанавливаем citus chttps://github.com/aeuge/citus-GKE
	Меняем пароль, на винде скрипт не работает, пришлось вручную менять ставить секрет
	Проблемы: 
	Не актуальные apiVersion
	1. https://github.com/aeuge/citus-GKE/blob/master/master.yaml#L27
	2. https://github.com/aeuge/citus-GKE/blob/master/workers.yaml#L14
	Актуальные здесь
	https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/
	https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
	Все остальное работает

4. Установка helm

	choco install kubernetes-helm
	
5. Ставим bitnami

	helm repo add bitnami https://charts.bitnami.com/bitnami
	helm install my-release bitnami/postgresql
	На виндос данная команда не работает так что пришлось пароль декодировать вручную 
	kubectl run my-release-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:11.9.0-debian-10-r48 --env="PGPASSWORD=kw9GiIuzjm" --command -- psql --host my-release-postgresql -U postgres -d postgres -p 5432
	

	