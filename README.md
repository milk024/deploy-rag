# deploy-rag

> 포트폴리오 공개용 스냅샷입니다. 실제 IP·도메인·조직명은 예시값(`10.0.0.x`, `example.ac.kr`, `myorg`)으로 바꿨고 비밀값은 포함하지 않습니다. 전체 소개는 [k8s-manager](https://github.com/milk024/k8s-manager) 를 보세요.

`rag-system` 네임스페이스 서비스의 **배포 상태 기준 저장소**. ArgoCD 가 이 저장소를 읽어 클러스터에 동기화한다.

| 앱 | 경로 | 소스 저장소 | 네임스페이스 |
|---|---|---|---|
| rag-server | `apps/rag-server` | `myorg/rag-server` | rag-system |
| rag-web | `apps/rag-web` | `myorg/rag-web` | rag-system |

## 흐름

```
소스 저장소 main 병합 → CI 빌드·이미지 push → 이 저장소 apps/<앱>/values.yaml 의 image.tag 갱신 커밋(rag-ci)
                                              → ArgoCD 가 감지해 동기화
```

## 규칙

- **main 직접 push 는 CI 계정 `rag-ci`(image.tag 갱신)만.** 그 외 변경은 PR + `rag-maintainers` 1명 승인.
- **비밀값을 커밋하지 않는다.** 아래는 클러스터에서 별도 관리하며 ArgoCD 대상이 아니다.
  - Secret `rag-app-secret`, `rag-postgres-secret`, 이미지 pull Secret
  - `rag-postgres`, `rag-pgadmin`, PVC(`rag-storage-pvc`, `rag-logs-pvc`), `rag-gateway`, `rag-http-to-https-redirect`
- 클러스터에서 `kubectl edit` / `helm upgrade` 로 직접 바꾸지 않는다. ArgoCD 가 이 저장소 기준으로 되돌린다.
- 이미지 레지스트리는 Gitea 컨테이너 레지스트리(`gpu.example.ac.kr/myorg/...`).

## ArgoCD

- `argocd/project-rag.yaml` — AppProject `rag` (이 저장소만 읽고 `rag-system` 에만 배포, 클러스터 범위·RBAC 리소스 금지)
- `argocd/app-*.yaml` — Application, 자동 동기화(prune + selfHeal)
- 접속: `kubectl port-forward -n argocd svc/argocd-server 8080:443` → https://localhost:8080

## 로컬 확인

```bash
helm template rag-server apps/rag-server -n rag-system | kubectl diff -n rag-system -f -
```

## 이력

- 2026-11-12: 소스 저장소 `k8s/` 차트 + 실제 클러스터 값(helm release)에서 이전. 이전 시점 `kubectl diff` = 0.
