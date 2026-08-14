# 후면 카메라 방위각 유도 수학적 증명 (Camera Heading Derivation)

이 문서는 `index.html` 파일의 `calculateCameraHeading` 함수 내 459-460번째 줄 코드(`east`, `north`)의 수학적 유도 과정과 회전 행렬 곱셈 순서의 수학적 의미를 정리한 설명서입니다.

---

## 1. 좌표계 정의 (Coordinate Systems)

### 1.1 지구 좌표계 (Earth Coordinate System)
- $X_{\text{earth}} = \text{East}$ (동쪽)
- $Y_{\text{earth}} = \text{North}$ (북쪽)
- $Z_{\text{earth}} = \text{Up}$ (하늘 방향)

### 1.2 기기 좌표계 (Device Coordinate System)
- $x_{\text{device}}$: 세로(Portrait) 상태 기준 오른쪽 (+x)
- $y_{\text{device}}$: 세로(Portrait) 상태 기준 상단 (+y)
- $z_{\text{device}}$: 화면 전면(사용자 방향, +z)

### 1.3 후면 카메라 시선 벡터 (Rear Camera Vector)
후면 카메라 렌즈는 스마트폰 뒷면(화면 안쪽)을 향하므로 시선 벡터 $\mathbf{v}_{\text{device}}$는 다음과 같습니다:

$$\mathbf{v}_{\text{device}} = \begin{bmatrix} 0 \\ 0 \\ -1 \end{bmatrix}$$

---

## 2. W3C 오일러 회전 행렬 (Rotation Matrices)

W3C Device Orientation 명세에 따른 회전 순서는 $Z \to X' \to Y''$ ($R_z(\alpha) \to R_x(\beta) \to R_y(\gamma)$)입니다.

지구 좌표계상에서의 카메라 벡터 $\mathbf{v}_{\text{earth}}$는 다음과 같이 정의됩니다:

$$\mathbf{v}_{\text{earth}} = R_z(\alpha) \cdot R_x(\beta) \cdot R_y(\gamma) \cdot \mathbf{v}_{\text{device}}$$

### 2.1 축별 회전 행렬

1. **$Z$-축 기준 $\alpha$ 회전 행렬 ($R_z(\alpha)$)**:
   $$R_z(\alpha) = \begin{bmatrix} \cos\alpha & -\sin\alpha & 0 \\ \sin\alpha & \cos\alpha & 0 \\ 0 & 0 & 1 \end{bmatrix}$$

2. **$X$-축 기준 $\beta$ 회전 행렬 ($R_x(\beta)$)**:
   $$R_x(\beta) = \begin{bmatrix} 1 & 0 & 0 \\ 0 & \cos\beta & -\sin\beta \\ 0 & \sin\beta & \cos\beta \end{bmatrix}$$

3. **$Y$-축 기준 $\gamma$ 회전 행렬 ($R_y(\gamma)$)**:
   $$R_y(\gamma) = \begin{bmatrix} \cos\gamma & 0 & \sin\gamma \\ 0 & 1 & 0 \\ -\sin\gamma & 0 & \cos\gamma \end{bmatrix}$$

### 2.2 회전 순서와 행렬 곱셈 순서의 관계 (Intrinsic vs Extrinsic Rotation)

W3C 명세에 명시된 물리적 회전 순서($Z \to X' \to Y''$)와 행렬 수식에서 $R_y(\gamma)$가 가장 우측(벡터 바로 옆)에 위치하는 이유는 **내인성(Intrinsic) 회전**과 **외인성(Extrinsic) 회전**의 좌표축 정의 차이 때문입니다.

| 구분 | **내인성 회전 (Intrinsic / Body-fixed)** | **외인성 변환 (Extrinsic / World-fixed)** |
| :--- | :--- | :--- |
| **좌표축 특성** | **회전에 따라 좌표축이 함께 따라 움직임** | **회전과 관계없이 절대 좌표축이 가만히 고정됨** |
| **기준 좌표계** | 스마트폰 자체에 붙어있는 기기 축 ($x, y, z$) | 지구 고정 절대 좌표계 ($X_{\text{East}}, Y_{\text{North}}, Z_{\text{Up}}$) |
| **해당 개념** | 물리적 기기 회전 (사용자/센서 관점) | 수학적 공간 복원 (행렬 곱셈 관점) |
| **작동 순서** | $Z(\alpha) \longrightarrow X'(\beta) \longrightarrow Y''(\gamma)$ | $R_y(\gamma) \longrightarrow R_x(\beta) \longrightarrow R_z(\alpha)$ (우측부터) |

- **물리적 기기 회전 (내인성 회전, Intrinsic)**:
  - 스마트폰은 1차로 $Z$-축($\alpha$)을 돌면 축이 공중에서 함께 꺾여 $X', Y'$가 되고, 2차 회전($\beta$)은 그렇게 **함께 꺾여버린 $X'$축**을 기준으로 진행됩니다.
- **수학적 공간 복원 (외인성 변환, Extrinsic)**:
  - 이미 최종 회전된 기기 좌표계의 벡터 $\mathbf{v}_{\text{device}}$를 **가만히 고정된 지구 절대 좌표계** $\mathbf{v}_{\text{earth}}$로 역산할 때는 **우측(벡터와 가장 가까운 행렬)부터 순서대로 작동**합니다.
  - 따라서 가장 나중에 회전된 $\gamma$ 회전 행렬($R_y$)부터 차례대로 풀어나가야 본래의 지구 좌표계가 복원됩니다.

$$\mathbf{v}_{\text{earth}} = \underbrace{R_z(\alpha)}_{\text{3차 연산}} \cdot \underbrace{R_x(\beta)}_{\text{2차 연산}} \cdot \underbrace{R_y(\gamma)}_{\text{1차 연산 (우측 우선)}} \cdot \mathbf{v}_{\text{device}}$$

---

## 3. 단계별 행렬 곱셈 유도 (Step-by-Step Derivation)

### 3.1 1단계: $R_y(\gamma) \mathbf{v}_{\text{device}}$ 연산

$$R_y(\gamma) \begin{bmatrix} 0 \\ 0 \\ -1 \end{bmatrix} = \begin{bmatrix} \cos\gamma & 0 & \sin\gamma \\ 0 & 1 & 0 \\ -\sin\gamma & 0 & \cos\gamma \end{bmatrix} \begin{bmatrix} 0 \\ 0 \\ -1 \end{bmatrix} = \begin{bmatrix} -\sin\gamma \\ 0 \\ -\cos\gamma \end{bmatrix}$$

### 3.2 2단계: $R_x(\beta) \left( R_y(\gamma) \mathbf{v}_{\text{device}} \right)$ 연산

$$R_x(\beta) \begin{bmatrix} -\sin\gamma \\ 0 \\ -\cos\gamma \end{bmatrix} = \begin{bmatrix} 1 & 0 & 0 \\ 0 & \cos\beta & -\sin\beta \\ 0 & \sin\beta & \cos\beta \end{bmatrix} \begin{bmatrix} -\sin\gamma \\ 0 \\ -\cos\gamma \end{bmatrix} = \begin{bmatrix} -\sin\gamma \\ \sin\beta \cos\gamma \\ -\cos\beta \cos\gamma \end{bmatrix}$$

### 3.3 3단계: $R_z(\alpha) \left( R_x(\beta) R_y(\gamma) \mathbf{v}_{\text{device}} \right)$ 연산

$$\mathbf{v}_{\text{earth}} = \begin{bmatrix} \cos\alpha & -\sin\alpha & 0 \\ \sin\alpha & \cos\alpha & 0 \\ 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} -\sin\gamma \\ \sin\beta \cos\gamma \\ -\cos\beta \cos\gamma \end{bmatrix}$$

---

## 4. 최종 성분 전개 및 코드 대조

행렬 곱셈 결과 벡터 $\mathbf{v}_{\text{earth}} = \begin{bmatrix} X_{\text{earth}} \\ Y_{\text{earth}} \\ Z_{\text{earth}} \end{bmatrix} = \begin{bmatrix} \text{East} \\ \text{North} \\ \text{Up} \end{bmatrix}$ 은 다음과 같이 성분별로 전개됩니다:

### 4.1 지구 동쪽 성분 ($X_{\text{earth}} = \text{East}$)
$$\text{East} = \cos\alpha \cdot (-\sin\gamma) + (-\sin\alpha) \cdot (\sin\beta \cos\gamma) + 0 \cdot (-\cos\beta \cos\gamma)$$
$$\mathbf{\text{East} = -\cos\alpha \sin\gamma - \sin\alpha \sin\beta \cos\gamma}$$

- **JavaScript 코드 (459번 줄)**:
  ```javascript
  const east = -(ca * sg + sa * sb * cg);
  ```

### 4.2 지구 북쪽 성분 ($Y_{\text{earth}} = \text{North}$)
$$\text{North} = \sin\alpha \cdot (-\sin\gamma) + \cos\alpha \cdot (\sin\beta \cos\gamma) + 0 \cdot (-\cos\beta \cos\gamma)$$
$$\mathbf{\text{North} = -\sin\alpha \sin\gamma + \cos\alpha \sin\beta \cos\gamma}$$

- **JavaScript 코드 (460번 줄)**:
  ```javascript
  const north = -sa * sg + ca * sb * cg;
  ```

### 4.3 지구 상향 성분 ($Z_{\text{earth}} = \text{Up}$)
$$\text{Up} = 0 \cdot (-\sin\gamma) + 0 \cdot (\sin\beta \cos\gamma) + 1 \cdot (-\cos\beta \cos\gamma)$$
$$\mathbf{\text{Up} = -\cos\beta \cos\gamma}$$

---

## 5. 최종 방위각(Azimuth) 환산

지평면 상에서 동쪽($\text{East}$)과 북쪽($\text{North}$) 성분을 2차원 극좌표계로 환산하여, 북쪽 기준 시계방향 방위각(0°=북, 90°=동, 180°=남, 270°=서)을 얻습니다.

$$\text{Heading} = \operatorname{atan2}(\text{East}, \text{North}) \times \frac{180}{\pi} \pmod{360}$$

- **JavaScript 코드 (471번 줄)**:
  ```javascript
  const headingDeg = Math.atan2(east, north) / degToRad;
  return normalizeHeading(headingDeg);
  ```
