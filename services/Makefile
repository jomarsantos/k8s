build:
	docker build -t <tag> .
	docker push <tag>

deploy:
	kubectl apply -f deploy.yml
	kubectl rollout restart deployment <deployment_name>

bd:
	make build
	make deploy
