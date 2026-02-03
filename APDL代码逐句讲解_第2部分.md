# APDL 代码逐句讲解 - 第2部分：等效热源与时间设置

## 第60-65行：等效热源计算

```apdl
!!-------------------------------------------!!
!! Heating Phase, loads appliation and solve !!
!!-------------------------------------------!!
```
**功能**：注释分隔符，标识加热阶段参数设置

```apdl
*SET, mm_, 0.25  !!等效热源系数，调节变形量!
```
**功能**：定义等效热源系数
- `mm_ = 0.25`（无量纲）
- **作用**：调节热源强度，从而控制变形量
- 这是一个经验系数，用于校准模型

```apdl
*SET, Q_AVE, mm_*p_*ap_/(2*R_*RS_HS_*pd_)   !!! equivalent heat source
```
**功能**：计算等效体热源强度
- **公式推导**：
  - 激光功率：`p_`
  - 粉末吸收系数：`ap_`
  - 有效功率：`p_ * ap_`
  - 热源体积：`2 * r_ * RS_HS_ * pd_`（近似为长方体）
  - 体热源强度：`Q_AVE = (mm_ * p_ * ap_) / (2 * r_ * RS_HS_ * pd_)`
- **单位**：W/μm³ = pW/μm³（在 μm-kg-s 单位系统中）
- **物理意义**：单位体积内的发热功率

```apdl
*SET, t_heating_, r_/v_    !! actual t_heating
```
**功能**：计算实际加热时间
- `t_heating_ = r_ / v_ = 75 / 950000 ≈ 7.89×10⁻⁵ s`
- **物理意义**：激光光斑通过一个光斑半径距离所需的时间

```apdl
*SET, t_cooling_, ((nint(PW/RS_HS_)+1 -1)*LOT_T_ + LOT_L_)  !! t_cooling of every layer
```
**功能**：计算每层的冷却时间
- `nint(PW/RS_HS_)`：打印件宽度内的扫描道数量（四舍五入）
  - `nint(2000/100) = 20` 道
- `(20+1-1) = 20`
- `t_cooling_ = 20 * 1000e-6 + 20 = 0.02 + 20 = 20.02 s`
- **组成**：扫描道间隔时间 + 层间冷却时间

---

## 第66-75行：加热阶段时间步设置

```apdl
*SET, Layer_TIME_0, Pre_time_
```
**功能**：设置第0层的起始时间
- `Layer_TIME_0 = Pre_time_ = 200e-6 s`

```apdl
*SET, i_step_sum_1, 5 + 9*0
```
**功能**：计算第1级时间步的累计步数
- `i_step_sum_1 = 5 + 0 = 5`
- 前5步使用 `CT_step_1_` 时间步长

```apdl
*SET, i_time_sum_1, i_step_sum_1*CT_step_1_
```
**功能**：计算第1级时间步的累计时间
- `i_time_sum_1 = 5 * 2e-6 = 10e-6 s = 10 μs`

```apdl
*SET, k_Time_heating_end, (t_heating_ - i_time_sum_1)/13
```
**功能**：计算加热阶段最后13步的时间步长
- `k_Time_heating_end = (7.89e-5 - 10e-6) / 13 ≈ 5.3e-6 s`
- **目的**：填充剩余的加热时间

```apdl
*DO, i, 2, 7, 1
    *SET, i_step_sum_%i%, 5 + 9*(i-1)
    *SET, i_time_sum_%i%, (i_step_sum_%i% - i_step_sum_%i-1%)*CT_step_%i%_ + i_time_sum_%i-1%
*ENDDO
```
**功能**：循环计算第2-7级时间步的累计步数和累计时间
- **i=2**：
  - `i_step_sum_2 = 5 + 9*1 = 14`
  - `i_time_sum_2 = (14-5)*CT_step_2_ + i_time_sum_1 = 9*10e-6 + 10e-6 = 100e-6 s`
- **i=3**：
  - `i_step_sum_3 = 5 + 9*2 = 23`
  - `i_time_sum_3 = (23-14)*CT_step_3_ + 100e-6 = 9*100e-6 + 100e-6 = 1000e-6 s`
- **依此类推**...
- **i=7**：
  - `i_step_sum_7 = 5 + 9*6 = 59`
  - `i_time_sum_7` 累计到较大值

---

## 第76-78行：定义总步数

```apdl
*SET, heating_step_sum_, 18
```
**功能**：定义加热阶段总步数
- `heating_step_sum_ = 18` 步

```apdl
*SET, cooling_step_sum_, 78
```
**功能**：定义冷却阶段总步数
- `cooling_step_sum_ = 78` 步

```apdl
*SET, step_sum_,heating_step_sum_ + cooling_step_sum_       ! 96 steps
```
**功能**：计算每层的总步数
- `step_sum_ = 18 + 78 = 96` 步
- **每层96步**：18步加热 + 78步冷却

---

## 第79-84行：逐层参数设置

```apdl
!--------------------------------!
! Layer-by-layer model parameter !
!--------------------------------!
```
**功能**：注释分隔符

```apdl
*DO, i_num, 1, n_mul_layer_, 1
    *SET, Sinle_layer_depth_%i_num%, Sinle_layer_depth
*ENDDO
```
**功能**：为每一层设置单层深度
- 循环：`i_num = 1` 到 `60`
- 创建变量：`Sinle_layer_depth_1`, `Sinle_layer_depth_2`, ..., `Sinle_layer_depth_60`
- 每个变量值都等于 `Sinle_layer_depth = 500 μm`
- **目的**：为每层单独设置参数，便于后续修改特定层的厚度

---

## 第85-88行：初始化累计变量

```apdl
*SET,t_cooling_0,0
```
**功能**：初始化第0层的冷却时间为0

```apdl
*SET,t_heating_cooling_0,0
```
**功能**：初始化第0层的加热+冷却总时间为0

```apdl
*SET,Layer_start_Time_0,0
```
**功能**：初始化第0层的起始时间为0

```apdl
*SET,Activation_height_0,0
```
**功能**：初始化第0层的激活高度为0

---

## 第89-98行：计算每层的时间和高度参数

```apdl
*DO, i_mul_layer, 1, n_mul_layer_, 1
    *SET, t_cooling_%i_mul_layer%, t_cooling_*Sinle_layer_depth_%i_mul_layer%/d_
*ENDDO
```
**功能**：计算每层的实际冷却时间
- 循环：`i_mul_layer = 1` 到 `60`
- `t_cooling_1 = t_cooling_ * Sinle_layer_depth_1 / d_`
- `t_cooling_1 = 20.02 * 500 / 40 = 250.25 s`
- **物理意义**：根据实际层厚与设定层厚的比例调整冷却时间

```apdl
*DO, i_mul_layer, 1, n_mul_layer_, 1
     *SET, t_heating_cooling_%i_mul_layer%, t_cooling_%i_mul_layer% + t_heating_   !! 
     *SET, Activation_height_%i_mul_layer%, Sinle_layer_depth_%i_mul_layer%+Activation_height_%i_mul_layer-1%      
     *SET, Layer_start_Time_%i_mul_layer%, Layer_start_Time_%i_mul_layer-1%+t_heating_cooling_%i_mul_layer-1%
     *SET, k_Time_cooling_end_%i_mul_layer%,(t_cooling_%i_mul_layer% - i_time_sum_7)/19
*ENDDO
```
**功能**：循环计算每层的关键时间和高度参数

**逐行解释**：

1. `t_heating_cooling_%i_mul_layer% = t_cooling_%i_mul_layer% + t_heating_`
   - 计算每层的加热+冷却总时间
   - 例如：`t_heating_cooling_1 = 250.25 + 7.89e-5 ≈ 250.25 s`

2. `Activation_height_%i_mul_layer% = Sinle_layer_depth_%i_mul_layer% + Activation_height_%i_mul_layer-1%`
   - 计算每层的累计激活高度
   - 例如：
     - `Activation_height_1 = 500 + 0 = 500 μm`
     - `Activation_height_2 = 500 + 500 = 1000 μm`
     - `Activation_height_60 = 30000 μm`

3. `Layer_start_Time_%i_mul_layer% = Layer_start_Time_%i_mul_layer-1% + t_heating_cooling_%i_mul_layer-1%`
   - 计算每层的起始时间（累计）
   - 例如：
     - `Layer_start_Time_1 = 0 + 0 = 0 s`
     - `Layer_start_Time_2 = 0 + 250.25 = 250.25 s`
     - `Layer_start_Time_3 = 250.25 + 250.25 = 500.5 s`

4. `k_Time_cooling_end_%i_mul_layer% = (t_cooling_%i_mul_layer% - i_time_sum_7) / 19`
   - 计算冷却阶段最后19步的时间步长
   - 用于填充剩余的冷却时间

---

## 第99-103行：定义边界条件参数

```apdl
!-----------------------------------------------!
! Predefine the perameter of boundary condition !
!-----------------------------------------------!
```
**功能**：注释分隔符

```apdl
*SET, HF_, 20                 ! Film Coefficient for surface ELEM (W/M^2 C)
```
**功能**：定义对流换热系数
- `HF_ = 20 W/(m²·°C)`
- 在 μm-kg-s 单位系统中需要转换

```apdl
*SET, TS_, 25
```
**功能**：定义环境温度
- `TS_ = 25 °C`

```apdl
*SET, TBottom_, 25
```
**功能**：定义基板底面温度
- `TBottom_ = 25 °C`
- 用于底面温度边界条件

---

**第2部分讲解完毕**，涵盖了：
1. 等效热源计算
2. 加热/冷却时间设置
3. 多级时间步长策略
4. 逐层参数计算
5. 边界条件参数

**关键理解**：
- 程序采用**96步/层**的策略（18步加热 + 78步冷却）
- 时间步长采用**对数递增**，从 2μs 到 1000s
- 每层的起始时间和激活高度都是**累计计算**的

下一部分将讲解材料属性定义（粉末、致密材料、基板）。
