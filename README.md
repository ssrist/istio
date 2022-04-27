###공통

namespace: {{ .Values.global.istioNamespace }}

네임스페이스 istio-system 사용하도록 고정

{{- range $key, $value := .Values.global.product }}

{{ $key }}: {{ $value | quote }}

{{- end }}

sdspaas.io/resource-kind: deployment

모든 deployment label에 추가


### istiod 차트 수정사항
--values.yaml 파일
tag 1.13.2 지정

traceSampling: 0.1 로 하향 조정 (0.1%)

nodeSelector
nodeAffinity
필요시 사용하려고 주석으로 추가

meshConfig:
  accessLogFile: /dev/stdout
로깅 옵션 추가

타임존, locale 추가

env:

  timezone: Asia/Seoul
  
  locale: ko_KR.UTF-8
  

--deployment.yaml 파일

name: PILOT_HTTP10

value: '1'

추가 (http1.0 프로토콜 관련)

1.13.2 버전 업데이트 되면서 필요없을수도 있음


타임존, locale 추가

- name: TZ

value: {{ .Values.env.timezone | quote }}

- name: LANG

value: {{ .Values.env.locale | quote }}





###ingress-gateway 차트(루트) 수정사항
Chart.yaml

apiversion: v2로 수정

dependencies 블록 추가

--values.yaml파일

prometheus, kiali, grafana, jaeger 블록 추가

global: pvc 블록 추가(persistence)



istio-ingressgateway nodePort 추가



type: NodePort로 변경



타임존, locale 설정 추가

global:

env:

timezone: Asia/Seoul

locale: ko_KR.UTF-8



istiod:

env:

timezone: Asia/Seoul

locale: ko_KR.UTF-8



global: 블록에
  product:
추가

--deployment.yaml 파일 속성 추가
- name: ISTIO_META_HTTP10
value: '"1"'

- name: TZ
value: {{ .Values.global.env.timezone | quote }}
- name: LANG
value: {{ .Values.global.env.locale | quote }}


###grafana, jaeger 설치 템플릿 추가

grafana 초기 로그인 admin/admin

###prometheus, kiali 파일
helm 블록 추가(설치여부, nodeselector 적용여부)
prometheus pvc 블록 추가
pvc.yaml 추가

env 추가
- name: TZ
          value: {{ .Values.kiali.env.timezone | quote }}
- name: LANG
          value: {{ .Values.kiali.env.locale | quote }}

env:

- name: TZ

value: {{ .Values.prometheus.env.timezone | quote }}

- name: LANG

value: {{ .Values.prometheus.env.locale | quote }}



kiali auth strategy: token으로 수정



TODO:

SCP 레지스트리 확인되면 이미지 태깅

values.yaml 파일들에 hub 경로 SCP 레지스트리로 수정

prometheus, kiali, grafana, jaeger 이미지 경로 수정

그 외 제약사항 적용



********테스트

/istio-1.13.2/manifests/charts/apps



alias k='kubectl'

k create ns istiotest

k create ns istio-app

k label ns istio-app istio-injection=enabled



helm install istio . -n istiotest



k apply -f bookinfo.yaml -n istio-app

k apply -f bookinfo-gateway.yaml -n istio-app

k apply -f gateway-kiali.yaml

k apply -f gateway-jaeger.yaml



http://localhost:30080/productpage


http://localhost:30080/kiali


kubectl get secret -n istio-system $(kubectl get sa kiali -n istio-system -o "jsonpath={.secrets[0].name}") -o jsonpath={.data.token} | base64 -d

http://localhost:30080/jaeger

prometheus와 grafana는 root url로 접속설정이라 port-forward로 접속해볼수 있음
예시 k port-forwart grafana 3000:3000 -n istio-system 


****uninstall

k delete -f bookinfo.yaml -n istio-app

k delete -f bookinfo-gateway.yaml -n istio-app

k delete -f gateway-kiali.yaml



/istio-1.13.2/manifests/charts/apps

helm uninstall istio -n istiotest



