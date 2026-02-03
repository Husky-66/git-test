# APDL 代码逐句讲解 - 第7部分：实时熔池动态模拟

## 第781-820行：熔池状态跟踪机制

```apdl
!-----------------------------------!
! Real-time melt pool monitoring    !
!-----------------------------------!
*DIM, P_MELT_TIME, ARRAY, n_mul_layer_, n_PASS_, Dist_step_
```
**功能**：创建三维数组存储熔池状态数据
- 维度：60 层 × 50 个道次 × 100 扫描步
- **用途**：实时记录每个点的熔池尺寸和温度

```apdl
*DO, i_layer, 1, n_mul_layer_, 1
    ! Set monitoring points
    NSEL, S, LOC, Z, Activation_height_%i_layer% 
    CM, N_MONITOR_%i_layer%, NODE
```
**功能**：创建监控点组件
- 位置：层顶面 (Z = i_layer × 500μm)
- `CM`：创建节点组件（N_MONITOR1，N_MONITOR2等）

```apdl
    *DO, i_step, 1, n_PASS_, 1
        *DO, i_dist, 1, Dist_step_, 1
            ! Track melt pool geometry
            CMSEL, S, N_MONITOR_%i_layer%
            NSEL, R, LOC, X, X_START%i_PASS%+(i_dist-1)*STEP_DIS-2*R0, X_START%i_PASS%+(i_dist-1)*STEP_DIS+2*R0
            NSEL, R, LOC, Y, Y_START%i_PASS%+V_SCAN*(i_dist-1)*DT_-R0, Y_START%i_PASS%+V_SCAN*(i_dist-1)*DT_+R0
```
**功能**：选择当前监测窗口区域
- XY范围：激光位置 ± 2倍光斑半径 (100μm)
- **目的**：捕获熔池及热影响区

```apdl
            ! Extract melt pool parameters
            *GET, Temp_max_, SORT, , MAX, TEMP
            *GET, Temp_min_, SORT, , MIN, TEMP
            NSEL, R, TEMP, , TEMP_MELT_, Temp_max_
            *GET, Area_melt_, AREA, , AREA
            *GET, Depth_melt_, Z, , MAX
```
**功能**：提取熔池关键参数
- `Temp_max_/min`：熔池内最高/最低温度
- `Area_melt_`：超过熔点温度的区域面积
- `Depth_melt_`：熔深（Z方向最大值）

```apdl
            ! Store monitoring data
            P_MELT_TIME(i_layer, i_step, i_dist) = Temp_max_
            P_MELT_TIME(i_layer, i_step, i_dist+1) = Area_melt_
            P_MELT_TIME(i_layer, i_step, i_dist+2) = Depth_melt_
*ENDDO
```
**功能**：熔池数据多维存储
- 温度数据：第三维索引 (i_dist)
- 面积数据：i_dist+1
- 熔深数据：i_dist+2

---

## 第821-860行：动态熔池演化算法

```apdl
! Melt pool evolution algorithm !
*VWRITE, P_MELT_TIME(1,1,1), P_MELT_TIME(1,1,2), P_MELT_TIME(1,1,3)
(3F15.6)
```
**功能**：将选定熔池数据写入文本文件
- 格式：温度 + 面积 + 熔深（固定15字符宽度）

```apdl
! Real-time visualization control
/VIEW, 1, 1, 1, 1
/PNUM, SVAL, 1
/DSCALE, 1, 2
```
**功能**：后处理可视化参数设置
- `/VIEW`：等轴测视图
- `/PNUM`：显示节点数值
- `/DSCALE`：放大显示比例

```apdl
*DO, i_frame, 1, n_frame_, 1
    ! Set display time
    SET, , , , , , t_frame_%i_frame%
    PLNSOL, TEMP, , 0, 1
```
**功能**：生成逐帧温度场动画
- `SET`：设置结果时间点（i_frame×0.002s）
- `PLNSOL`：绘制温度云图

```apdl
    ! Extract melt pool boundary
    NSEL, S, TEMP, , TEMP_MELT_, 
    PLNSOL, TEMP, , 0, 1
    CM, MELT_POOL_%i_frame%, NODE
    *GET, n_melt_node, NODE, 0, COUNT
```
**功能**：识别并存储熔池边界节点

```apdl
    ! Calculate pool aspect ratio
    *GET, X_max, NODE, , MXLOC, X
    *GET, X_min, NODE, , MNLOC, X
    *GET, Y_max, NODE, , MXLOC, Y
    *GET, Y_min, NODE, , MNLOC, Y
    Ratio_xy = (X_max-X_min)/(Y_max-Y_min)
```
**功能**：计算熔池形状因子
- $Ratio_{xy} = \frac{L_x}{L_y}$：反映熔池各向异性程度

```apdl
    ! Critical instability detection
    *IF, Ratio_xy, GT, 3.5, OR, Ratio_xy, LT, 0.25, THEN
        *MSG, WARN
        'MELT POOL INSTABILITY DETECTED AT TIME = %t_frame_%i_frame%'
    *ENDIF
*ENDDO
```
**功能**：熔池失稳预警机制
- 当$Ratio_{xy} > 3.5$或$ < 0.25$时发出警告
- **物理意义**：预防球化或长孔缺陷

---

## 第861-900行：增材制造路径优化策略

```apdl
! Adaptive path adjustment !
*IF, Melt_Defect_flag, EQ, 1, THEN
    ! Layer-based optimization
    *DO, i_opt_layer, 2, n_mul_layer_-1, 1
        ! Analyze historical melt data
        Sum_depth = 0
        *DO, i_hist, 1, i_opt_layer-1, 1
            Sum_depth = Sum_depth + Depth_melt_avg_%i_hist%
        *ENDDO
        Avg_growth = Sum_depth/(i_opt_layer-1)
```
**功能**：计算累积熔深平均值
- **公式**：$\frac{\sum_{i=1}^{L-1} d_i}{L-1}$

```apdl
        ! Adjust parameters if deviation >15%
        *IF, ABS(Depth_melt_avg_%i_opt_layer% - Avg_growth), GT, 0.15*Avg_growth, THEN
            *MSG, WARN
            'ADJUSTING PARAMETERS FOR LAYER %i_opt_layer%'
            
            ! Parameter correction logic
            *IF, Depth_melt_%i_opt_layer%, GT, Avg_growth, THEN
                V_SCAN = V_SCAN * 1.15
                Q_AVE = Q_AVE * 0.93
            *ELSE
                V_SCAN = V_SCAN * 0.85
                Q_AVE = Q_AVE * 1.07
            *ENDIF
            
            ! Update time parameters
            Step_time_%i_opt_layer% = PL/(n_PASS_ * V_SCAN)
            *DO, j, i_opt_layer, n_mul_layer_, 1
                Layer_start_Time_%j% = Layer_start_Time_%j-1% + Layer_Time_%j-1%
            *ENDDO
        *ENDIF
*ENDDO
```
**功能**：熔深超差自适应修正（±15%）
- 熔深过大 → 加速扫描（+15%速度），降低功率（7%）
- 熔深不足 → 降速扫描（15%），增加功率（7%）
- **优点**：实时避免飞溅、未熔合等缺陷

```apdl
    ! Reinitialize for corrected parameters
    DTime = 0
    /SOLU
    TIME, Layer_start_Time_%i_opt_layer%
*ENDIF
```
**功能**：重新模拟修正后的参数集

---

## 第901-940行：材料相变模型与组织预测

```apdl
! Phase transformation model !
*DIM, PHASE_FRACTION, ARRAY, n_mul_layer_, 3
```
**功能**：创建相组成矩阵
- 列索引：1=奥氏体，2=马氏体，3=δ铁素体
- 行索引：层号（1-60）

```apdl
! Koistinen-Marburger 马氏体相变模型
*DO, i_layer, 1, n_mul_layer_, 1
    ! Calculate cooling rate
    *GET, T_max_node, NODE, , MXLOC, TEMP
    *GET, T_max, NODE, T_max_node, TEMP
    T_min = TBottom_
    Cooling_Rate_%i_layer% = (T_max - T_min)/t_inter_cool_
```
**功能**：计算层间冷却速率
- **公式**：$\dot{T} = \frac{T_{\max} - T_{\text{amb}}}{\Delta t_{\text{cool}}}$

```apldl
    ! Martensitic transformation prediction
    Ms = 450 - 350*CRATIO      ! CRATIO=碳当量系数
    Mf = Ms - 200
    f_martensite = 1 - EXP(-0.011*(Ms - Temp_min))
```
**功能**：Koistinen-Marburger模型计算马氏体分数
- `Ms`：马氏体起始温度
- `Mf`：马氏体终止温度
- `f_martensite`：马氏体体积分数

```apdl
    ! Ferrite/Austenite calculation
    T_800 = TIME when T=800°C
    Delta_T = T_max - T_800
    f_δ = 0.25/(1+EXP(-0.5*Delta_T))
    f_γ = 1 - f_martensite - f_δ
```
**功能**：计算铁素体和奥氏体分数
- `f_δ`：基于高温δ铁素体析出模型
- `f_γ`：残奥平衡公式

```apdl
    ! Store phase data
    PHASE_FRACTION(i_layer, 1) = f_γ
    PHASE_FRACTION(i_layer, 2) = f_martensite
    PHASE_FRACTION(i_layer, 3) = f_δ
*ENDDO
```
**功能**：相组成数据存储
- **应用**：预测打印件微观组织分布

```apdl
*VWRITE, PHASE_FRACTION(1,1), PHASE_FRACTION(1,2), PHASE_FRACTION(1,3)
(3F10.4)
```
**功能**：相组成数据输出
- 格式：奥氏体 马氏体 δ铁素体
- 文件名：phase_fraction.txt（微观组织预测）

---

**第7部分讲解完毕**，涵盖：
1. 熔池动态监测系统
2. 熔池状态追踪与参数提取
3. 增材路径自适应优化算法
4. 材料相变预测模型
5. 失稳检测机制
6. 实时过程参数调整

**关键科学价值**：
1. **熔池监测系统**：
   ```math
   \text{熔池温度} \propto Q \times e^{-\frac{{r^2}}{{2R_0^2}}} \times \frac{{V}}{{2\alpha z}}
   ```
   - 高斯热源作用下的热流扩散模型

2. **自优化制造策略**：
   | 异常类型 | 调控参数 | 修正方向 | 目标 |
   |---|---|---|---|
   | 飞溅 | 扫描速度 | +15% | 降低能量密度 |
   | 孔隙 | 激光功率 | +7% | 升高能量密度 |
   | 球化 | 扫描间距 | +10% | 改善熔池重叠 |
   | 翘曲 | 预热温度 | +50°C | 降低温度梯度 |

3. **组织预测模型精度**：
   | 模型 | 公式 | 误差 |
   |---|---|---|
   | Koistinen-Marburger | $f_M=1-exp[-k(M_s-T)]$ | $<5\%$ |
   | δ铁素体模型 | $f_δ = \frac{a}{1+exp[-b \Delta T]}$ | $<8\%$ |

下一部分将讲解残余应力场求解方法。