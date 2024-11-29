
# Kubernetes Service 개념 및 접근 방법

Kubernetes Service는 Pod의 네트워크 접근을 관리하고, 로드 밸런싱 및 네트워크 엔드포인트 관리를 제공합니다.  
이 문서에서는 Kubernetes Service의 개념, 종류, 각 접근 방법, 그리고 예시 매니페스토 파일과 함께 설명합니다.

---

## 1. Kubernetes Service란?

- **Pod 간의 통신**: Pod는 생성과 삭제가 반복되며 IP가 동적으로 변경됩니다. Service는 이러한 Pod의 네트워크를 안정적으로 연결합니다.
- **로드 밸런싱**: Service는 동일한 Label Selector를 공유하는 여러 Pod 간의 트래픽을 로드 밸런싱합니다.
- **고정 IP 제공**: Service는 Cluster 내부와 외부에서 Pod에 접근할 수 있는 안정적인 네트워크 인터페이스를 제공합니다.

---

## 2. Kubernetes Service 종류 및 개념

1. **ClusterIP (기본값)**:
   - 클러스터 내부에서만 접근 가능한 내부 IP를 제공.
   - 외부 네트워크에서 접근할 수 없음.
   - 주로 클러스터 내부 통신에 사용.

2. **NodePort**:
   - 각 노드의 특정 포트를 열어 외부 네트워크에서 접근 가능.
   - 노드의 IP와 지정된 포트를 통해 Service에 접근.
   - 포트 범위: `30000-32767`.

3. **LoadBalancer**:
   - 클라우드 제공자의 로드 밸런서를 사용하여 외부 트래픽을 분산.
   - 클라우드 플랫폼(AWS, GCP, Azure 등)에서 지원.
   - 클러스터 외부의 공인 IP를 통해 접근.

4. **ExternalName**:
   - 클러스터 외부의 DNS 이름을 내부에서 참조 가능.
   - IP 대신 DNS 이름으로 트래픽을 라우팅.

---

## 3. Service 종류별 접근 방법 (URL)

1. **ClusterIP**:
   - 접근 가능: 클러스터 내부에서만.
   - URL: `http://<Service 이름>.<Namespace>.svc.cluster.local:<포트>`

2. **NodePort**:
   - 접근 가능: 외부 네트워크에서.
   - URL: `http://<노드 IP>:<NodePort>`

3. **LoadBalancer**:
   - 접근 가능: 외부 네트워크에서.
   - URL: `http://<외부 IP>:<포트>`

4. **ExternalName**:
   - 접근 가능: 클러스터 내부에서 외부 DNS 이름으로.
   - URL: `http://<ExternalName>`

---

## 4. 예시 매니페스토 파일

### 4.1 ClusterIP Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: example-clusterip
  namespace: default
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```

> **각주**:
> - `port`: Service가 노출하는 포트.
> - `targetPort`: 연결될 Pod의 포트.

---

### 4.2 NodePort Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: example-nodeport
  namespace: default
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30001
  type: NodePort
```

> **각주**:
> - `nodePort`: 고정된 외부 포트 번호. 30000-32767 범위 내에서 설정.

---

### 4.3 LoadBalancer Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: example-loadbalancer
  namespace: default
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

> **각주**:
> - 클라우드 플랫폼에 따라 로드 밸런서 생성 및 외부 IP 할당.

---

### 4.4 ExternalName Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: example-externalname
  namespace: default
spec:
  type: ExternalName
  externalName: example.com
```

> **각주**:
> - `externalName`: 클러스터 외부 DNS 이름.

---

## 5. 결론

Kubernetes Service는 네트워크 복잡성을 줄이고 안정적인 Pod 접근을 제공합니다.  
각 Service 유형의 장단점과 사용 사례를 이해하면 애플리케이션의 요구 사항에 적합한 네트워크 구성을 할 수 있습니다.
