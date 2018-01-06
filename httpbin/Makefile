.PHONY: all
all: tls.key namespace secret apply

tls.key:
	openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./tls.key -out ./tls.crt -subj "/CN=httpbin.127.0.0.1.xip.io"

.PHONY: namespace
namespace:
	kubectl apply -f ./config/kubernetes/dev/namespaces/httpbin.yaml

.PHONY: secret
secret:
	kubectl -n httpbin get secret httpbin-tls || kubectl -n httpbin create secret tls httpbin-tls --key ./tls.key --cert ./tls.crt

.PHONY: apply
apply:
	kubectl apply -Rf ./config/kubernetes/dev
