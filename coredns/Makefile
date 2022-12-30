.PHONY: build
build:
	docker build -t .../...:latest .

.PHONY: build-and-push
build-and-push: build push

.PHONY: run
run:
	docker run -p 5356:5356/tcp -p 5356:5356/udp -d --name dns1 .../...:latest

.PHONY: push
push:
	docker image push ../..:latest