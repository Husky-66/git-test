# APDL 代码逐句讲解 - 第1部分：初始化与参数定义

## 第1-3行：获取作业名并设置输出文件

```apdl
*GET, JBN_,  ACTIVE, 0, JOBNAM  ! get jobname
```
**功能**：获取当前 ANSYS 作业的名称，存储到变量 `JBN_` 中
- `*GET`：APDL 参数提取命令
- `ACTIVE, 0, JOBNAM`：从活动数据库中获取作业名

```apdl
/OUTPUT, %JBN_%_Output_Window_Relocate_LAM, DAT ! output
```
**功能**：将所有后续输出重定向到文件
- `/OUTPUT`：输出控制命令
- `%JBN_%`：使用作业名作为文件名前缀
- `_Output_Window_Relocate_LAM.DAT`：输出文件后缀
- 例如：如果作业名是 "test"，则输出到 "test_Output_Window_Relocate_LAM.DAT"

---

## 第4-6行：注释分隔符

```apdl
!------------------------------!
! Predefine process parameters ! 
!------------------------------!
```
**功能**：注释，标识下面是工艺参数定义部分

---

## 第7-14行：定义激光扫描工艺参数

```apdl
*SET, v_,   950*1e3     ! actual scan speed
```
**功能**：定义激光扫描速度
- `*SET`：设置参数值
- `v_ = 950*1e3 = 950,000 μm/s = 950 mm/s`
- 单位系统：μm-kg-s

```apdl
*SET, p_,   300e12       ! Laser power
```
**功能**：定义激光功率
- `p_ = 300e12 = 300×10¹² pW = 300 W`
- 单位转换：W = kg·m²/s³ → pW (pico-watt in μm system)

```apdl
*SET, d_,   40          ! The thickness of powder bed
```
**功能**：定义粉末床单层厚度
- `d_ = 40 μm`（微米）

```apdl
*SET, pd_,  40          ! Penetration Depth of the powder layer
```
**功能**：定义激光穿透深度
- `pd_ = 40 μm`
- 用于计算等效热源强度

```apdl
*SET, r_,   75          ! Laser spot radius
```
**功能**：定义激光光斑半径
- `r_ = 75 μm`

```apdl
*SET, RS_HS_, 100        ! Hatch Spacing for Raster Scanning
```
**功能**：定义光栅扫描的扫描间距
- `RS_HS_ = 100 μm`
- Hatch Spacing：相邻扫描道之间的距离

```apdl
*SET, CS_HS_, 60       ! Hatch Spacing for Contour Scanning
```
**功能**：定义轮廓扫描的扫描间距
- `CS_HS_ = 60 μm`
- 轮廓扫描通常间距更小，以获得更好的表面质量

---

## 第15-20行：定义激光吸收系数

```apdl
!----------------------------!
! Predefine  laser parametes !
!----------------------------!
```
**功能**：注释分隔符

```apdl
*SET, ap_,  0.42        ! Metal powder absorption coefficient@1.06 micro
```
**功能**：定义金属粉末对激光的吸收系数
- `ap_ = 0.42`（无量纲）
- @1.06 μm：激光波长（Nd:YAG 激光器）

```apdl
*SET, al_,  0.42        ! Metal liquid absorption coefficient@1.06 micro
```
**功能**：定义熔融金属对激光的吸收系数
- `al_ = 0.42`

```apdl
*SET, ak_, 0.9          ! Absorption when laser enters into the key hole.
```
**功能**：定义激光进入小孔（keyhole）时的吸收系数
- `ak_ = 0.9`
- Keyhole：激光功率密度极高时形成的蒸发孔

---

## 第21-35行：定义几何参数

```apdl
!--------------------------!
! Predefine geo parameters !
!--------------------------!
```
**功能**：注释分隔符

```apdl
*SET, phrb, 1000                ! element height of powder bed
```
**功能**：定义粉末床单元高度
- `phrb = 1000 μm = 1 mm`

```apdl
*SET, Height, 30000             ! the height of the part
```
**功能**：定义打印件总高度
- `Height = 30000 μm = 30 mm`

```apdl
*SET, Sub_length, 70000         ! the length of the substrate
```
**功能**：定义基板长度（X方向）
- `Sub_length = 70000 μm = 70 mm`

```apdl
*SET, Sub_width, 50000          ! the width of the substrate
```
**功能**：定义基板宽度（Y方向）
- `Sub_width = 50000 μm = 50 mm`

```apdl
*SET, Sub_height, -30000        ! the height of the substrate
```
**功能**：定义基板高度（Z方向，负值表示在零平面以下）
- `Sub_height = -30000 μm = -30 mm`

```apdl
*SET, PL, 50000                 ! the length of the part
```
**功能**：定义打印件长度
- `PL = 50000 μm = 50 mm`

```apdl
*SET, PW, 2000                 ! the width of the part
```
**功能**：定义打印件宽度
- `PW = 2000 μm = 2 mm`

```apdl
*SET, n_mul_layer_, 60          ! total number of layers
```
**功能**：定义总层数
- `n_mul_layer_ = 60` 层

```apdl
*SET, rb, Height/n_mul_layer_   ! basic mesh size
```
**功能**：计算基本网格尺寸
- `rb = 30000/60 = 500 μm`
- 每层的厚度作为基本网格尺寸

```apdl
*SET, Sinle_layer_depth, Height/n_mul_layer_
```
**功能**：计算单层深度
- `Sinle_layer_depth = 500 μm`

---

## 第36-48行：定义网格划分参数

```apdl
!mesh
*SET, NDIV_XY, (Sub_length-PL)/2/rb
```
**功能**：计算 XY 方向的网格划分数
- `NDIV_XY = (70000-50000)/2/500 = 20`
- 基板边缘区域的单元数

```apdl
*SET, NDIV_Vol, 8
```
**功能**：定义体积区域的网格划分数
- `NDIV_Vol = 8`

```apdl
*SET, NDIV_Z_B, 6
```
**功能**：定义 Z 方向 B 区域的网格划分数
- `NDIV_Z_B = 6`

```apdl
*SET, NDIV_Z_C, 4
```
**功能**：定义 Z 方向 C 区域的网格划分数
- `NDIV_Z_C = 4`

```apdl
*SET, NDIV_Z_D, 3
```
**功能**：定义 Z 方向 D 区域的网格划分数
- `NDIV_Z_D = 3`

```apdl
*SET, SR_XY, 1.5
```
**功能**：定义 XY 方向的网格间距比
- `SR_XY = 1.5`（Space Ratio）
- 网格从中心向边缘逐渐变大

```apdl
*SET, SR_Vol, 12.9519
```
**功能**：定义体积区域的网格间距比
- `SR_Vol = 12.9519`

```apdl
*SET, SR_Z_B, 3.5987
```
**功能**：定义 Z 方向 B 区域的网格间距比

```apdl
*SET, SR_Z_C, 1
```
**功能**：定义 Z 方向 C 区域的网格间距比（均匀网格）

```apdl
*SET, SR_Z_D, 1.5244
```
**功能**：定义 Z 方向 D 区域的网格间距比

---

## 第49-54行：定义激光关闭时间

```apdl
!-------------------------------------------------------!
! Defines laser off time during laser moving processing !
!-------------------------------------------------------!
```
**功能**：注释分隔符

```apdl
*SET, Pre_time_, 200e-6     ! the initial time before laser on
```
**功能**：定义激光开启前的初始时间
- `Pre_time_ = 200×10⁻⁶ s = 0.0002 s`

```apdl
*SET, LOT_T_, 1000e-6       ! Set the LASER OFF TIME between TRACKS
```
**功能**：定义扫描道之间的激光关闭时间
- `LOT_T_ = 1000×10⁻⁶ s = 0.001 s = 1 ms`

```apdl
*SET, LOT_L_, 20            ! the LASER OFF TIME between LAYERS
```
**功能**：定义层与层之间的激光关闭时间
- `LOT_L_ = 20 s`

---

## 第55-59行：定义时间步长

```apdl
*SET, CT_step_1_, 2e-6      !!! set CT_step
```
**功能**：定义第1级时间步长
- `CT_step_1_ = 2×10⁻⁶ s = 2 μs`

```apdl
*DO, i, 2, 10
  *SET, CT_step_%i%_, 10**(i-7)
*ENDDO
```
**功能**：循环定义第2-10级时间步长
- `*DO, i, 2, 10`：循环从 i=2 到 i=10
- `CT_step_2_ = 10^(2-7) = 10^(-5) = 10 μs`
- `CT_step_3_ = 10^(3-7) = 10^(-4) = 100 μs`
- `CT_step_4_ = 10^(-3) = 1 ms`
- ...
- `CT_step_10_ = 10^3 = 1000 s`
- **目的**：采用对数递增的时间步长，加热阶段用小步长，冷却阶段用大步长

---

**第1部分讲解完毕**，涵盖了：
1. 输出设置
2. 工艺参数定义
3. 几何参数定义
4. 网格参数定义
5. 时间控制参数

下一部分将讲解等效热源计算和加热/冷却阶段设置。
