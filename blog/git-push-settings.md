# `push.default` vs `remote.pushDefault` 설정 비교

좋은 질문입니다! 둘 다 `git push`의 기본 동작을 제어하지만, **서로 다른 차원의 설정**입니다.

---

## 1. `push.default` — "어떤 브랜치를 push할 것인가"

`git push`를 리모트 브랜치 지정 없이 실행했을 때, **어떤 브랜치를 대상으로 push할지** 결정합니다.

| 값 | 동작 |
|---|---|
| `nothing` | 항상 명시해야 함 (가장 안전) |
| `current` | 현재 브랜치와 **같은 이름**의 리모트 브랜치로 push |
| `upstream` (= `tracking`) | 현재 브랜치가 **tracking하는 리모트 브랜치**로 push (이름이 달라도 됨) |
| `simple` | `upstream`과 비슷하지만, 로컬과 리모트 브랜치 **이름이 같을 때만** push (Git 2.0+ 기본값) |
| `matching` | 로컬과 리모트에 **같은 이름의 브랜치가 모두 있는 것 전부** push (Git 1.x 기본값, 위험!) |

> ★ **Insight**
>
> **1. 현재 설정이 `current`인 것은 편의성 중심의 설정입니다.** tracking 설정 없이도 바로 push할 수 있어서 편하지만, 리모트가 여러 개일 때 의도하지 않은 곳에 push할 가능성이 있습니다.
>
> **2. `simple`이 Git 기본값인 이유는** `upstream`의 유연함과 안전성의 균형을 잡기 위해서입니다. tracking이 설정되어 있어야 하고, 이름도 같아야 하므로 실수를 방지합니다.
>
> **3. `matching`은 과거의 유물입니다.** 예를 들어 `main`, `dev`, `feature` 브랜치가 모두 리모트에 있으면 세 개 다 한꺼번에 push해버립니다. 이 때문에 Git 2.0에서 기본값이 `simple`로 변경되었습니다.

---

## 2. `remote.pushDefault` — "어떤 리모트에 push할 것인가"

리모트가 여러 개일 때, **어느 리모트 서버로 push할지** 결정합니다.

```bash
# 전역 설정
git config --global remote.pushDefault origin

# 이 저장소만
git config remote.pushDefault upstream
```

예를 들어 fork 기반 워크플로우에서:
- `origin` → 내 fork
- `upstream` → 원본 저장소

이 경우 `remote.pushDefault`를 `origin`으로 설정하면 항상 내 fork로 push하게 됩니다.

---

## 두 설정의 관계 정리

`git push`를 인자 없이 실행하면 Git은 두 가지를 결정해야 합니다:

```
git push [어디로?] [무엇을?]
         ↑              ↑
   remote.pushDefault   push.default
   (리모트 선택)         (브랜치 선택)
```

구체적인 결정 흐름은 이렇습니다:

```
[리모트 결정]
  1. 브랜치에 tracking 설정이 있으면 → 그 리모트
  2. remote.pushDefault가 있으면 → 그 값
  3. 둘 다 없으면 → origin (하드코딩 기본값)

[브랜치 결정]
  push.default 설정에 따라 위 표대로 동작
```

> ★ **Insight**
>
> **1. 현재 이 저장소의 상황을 대입하면:** `week1` 브랜치에는 tracking이 없고, `remote.pushDefault`도 미설정이므로 리모트는 `origin`으로 결정됩니다. 그리고 `push.default = current`이므로 `origin/week1`으로 push됩니다.
>
> **2. Fork 워크플로우에서 권장 설정:**
> ```bash
> git config remote.pushDefault origin    # 항상 내 fork로 push
> git remote add upstream <원본 URL>       # pull은 upstream에서
> ```
> 이렇게 하면 실수로 원본 저장소에 push하는 사고를 방지할 수 있습니다.
