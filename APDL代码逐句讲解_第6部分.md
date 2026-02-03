# APDL 代码逐句讲解 - 第6部分：求解器设置与热分析

## 第641-680行：求解器初始化设置

```apdl
/SOLU
ANTYPE, TRANS
```
**功能**：进入求解器并指定瞬态分析
- `ANTYPE, TRANS`：瞬态热分析（温度随时间变化）

```apdl
SOLCONTROL, ON
CNVTOL, HEAT, , 1e-6
```
**功能**：求解精度控制
- `SOLCONTROL, ON`：使用增强的非线性算法
- `CNVTOL, HEAT`：热自由度收敛容差设为1×10⁻⁶

```apdl
NROPT, FULL, ,ON
AUTOTS, ON
```
**功能**：非线性求解选项
- `NROPT, FULL`：使用完全牛顿-拉夫森法
- `AUTOTS, ON`：自动时间步长控制，提高求解效率

```apdl
! Define time step control !
KBC, 0
```
**功能**：设置斜坡加载（非阶跃加载）
- **物理意义**：载荷随时间平滑变化

```apdl
DELTIM, DT_STEP_MIN_, DT_STEP_MAX_, DT_STEP_MIN_
OUTRES, ALL, DT_OUT_
```
**功能**：时间步长控制和输出设置
- `DELTIM`：初始时间步长 = 10⁻⁶ s
- 最小步长 = 10⁻⁷ s，最大步长 = 10⁻¹ s
- `OUTRES, ALL, 10`：输出所有变量，每10步保存一次结果

```apdl
TIMINT, OFF, THERMAL
TIME, DTI_
SOLVE
TIMINT, ON, THERMAL
```
**功能**：执行初始静热分析
- `TIME, DTI_=10⁻⁶s`：模拟初始热平衡状态
- **目的**：消除初始不稳定状态

```apdl
! Define load step files !
RESCONTRL, DEFINE, ALL, 1, 50
```
**功能**：控制结果文件写入频率
- `ALL, 1`：每个子步都写结果
- `50`：限制结果文件数（自动删除旧文件）

---

## 第681-720行：热源加载与生死单元控制

```apdl
!-------------------Heat source moving-----------------!
! Define Gaussian distribution heat source
HFUNC, Q_VOLU, Q_VOLU
```
**功能**：加载表格定义的体热源
- `Q_VOLU`：第5部分定义的TABLE数组

```apdl
*DO, i_layer, 1, n_mul_layer_, 1
    ! Set activation time
    TIME, Layer_start_Time_%i_layer%
    *SET, DTime, 0
    *SET, DT_LAYER, Step_time_%i_layer%
```
**功能**：开始新层分析循环
- `TIME`：设置当前层开始时间
- `DT_LAYER` = 0.01 s：默认层分析时间步长

```apdl
    ! Activate elements
    ALLSEL, ALL
    EKILL, E_GRA%i_layer%
    ESEL, S, LIVE
    EALIVE, E_GRA%i_layer%
    *SET, ELEMENT_(i_layer), 1
```
**功能**：激活当前层单元
- `EKILL`：杀死粉末区单元
- `EALIVE`：激活当前层单元（变为实体）
- **转变机理**：粉末→熔融→凝固

```apdl
    *DO, i_step, 1, n_PASS_, 1
        ! Move heat flux
        *DO, i_dist, 1, Dist_step_, 1
            ! Calculate laser position
            X_LAS = X_START%i_PASS% + (i_dist-1)*STEP_DIS
            Y_LAS = Y_START%i_PASS% + V_SCAN*DTime
```
**功能**：激光路径计算循环
- `Dist_step_ = 100`：每个路径分成100段
- `X_LAS, Y_LAS`：激光中心实时坐标

```apdl
            ! Apply heat flux to elements
            NSEL, S, LOC, X, X_LAS-R0, X_LAS+R0
            NSEL, R, LOC, Y, Y_LAS-R0, Y_LAS+R0
            NSEL, R, LOC, Z, Activation_height_%i_layer-1%, Activation_height_%i_layer%
            BF, ALL, HGEN, %Q_VOLU%
```
**功能**：在激光作用区域施加热生成载荷
- `R0 = 50μm`：激光光斑半径
- `BF`：施加体热源（单位体积热生成率）
- `%Q_VOLU%`：温度相关热源强度

```apdl
            ! Set time increment
            TIME, Layer_start_Time_%i_layer% + DTime
            DELTIM, Calc_Dt_, , , 
            SOLVE
            DTime = DTime + Calc_Dt_
*ENDDO
```
**功能**：时间步进求解
- `Calc_Dt_ = 10⁻⁴s`：激光移动步长时间
- 累计时间`DTime`：跟踪层内已分析时间

---

## 第721-760行：材料转换与冷却阶段

```apdl
        ! Material conversion phase
        *IF, DTime, GT, MATERIAL_CHANGE_TIME(i_layer), THEN
            CMSEL, S, E_GRA%i_layer%
            EPLOT
            MPCHG, 2, ALL
        *ENDIF
```
**功能**：检测并执行材料转换
- 对比当前时间和材料转换时间参数
- `MPCHG, 2`：将激活区域转换为致密材料（MAT 2）

```apdl
    ! Cooling between passes（道次间冷却）
    TIME, Layer_start_Time_%i_layer% + DTime
    DELTIM, t_COOL_, , ,1
    KBC, 0
    SOLVE
*ENDDO
```
**功能**：道次间冷却分析
- `t_COOL_ = 0.004s`：层内道次间冷却时间
- 斜坡加载方式：`KBC, 0`

```apdl
    ! Inter-layer cooling（层间冷却）
    TIME, Layer_Start_Time_%i_layer% + Layer_Time_%i_layer%
    DELTIM, t_inter_cool_, t_max_, 
    SOLVE
*ENDDO    
```
**功能**：层间冷却分析
- `t_inter_cool_ = 0.05s`：层间冷却时间
- **物理意义**：模拟当前层完全沉积后的冷却

---

## 第761-780行：最终冷却阶段

```apdl
! Final cooling phase
TIME, T_total_
DELTIM, t_cool_, , , , 
KBC, 0
SOLVE
```
**功能**：最终冷却分析
- `T_total_ = 60s`：总分析时间
- `t_cool_ = t_inter_cool_`：冷却时间步长

```apdl
! Save final results
ALLSEL, ALL
SAVE, FINAL_RESULT, DB
FINISH
```
**功能**：保存并退出求解器
- `SAVE`：结果文件存入数据库
- **文件名**：FINAL_RESULT.DB（热分析结果）

---

**第6部分讲解完毕**，涵盖：
1. 求解器初始化与非线性控制
2. 时间步长策略
3. 体热源移动加载实现
4. 生死单元激活流程
5. 材料转换机制
6. 道次间及层间冷却模拟
7. 结果保存方法

**关键技术要点**：
1. **移动热源实现**：
   - 双重嵌套循环：层循环 → 道次循环 → 扫描步循环
   - 动态计算激光位置 ($X_{LAS}$, $Y_{LAS}$)
   - 局部节点选择施加体热源

2. **材料转换策略**：
   ```math
   t_{\text{转换}} = \text{Layer\_start\_Time} + \frac{t_{\text{heating}}}{2}
   ```
   - 温度场达到半程时转换材料属性

3. **多尺度时间步长**：
   | 阶段 | 步长 | 物理意义 |
   |---|---|---|
   | 激光扫描 | 1×10⁻⁴ s | 熔池动态过程 |
   | 道次间冷却 | 0.004 s | 短线冷却 |
   | 层间冷却 | 0.05 s | 较长冷却 |
   | 最终冷却 | 0.05 s | 整体冷却 |

4. **结果输出优化**：
   - `RESCONTRL`限制结果文件数量
   - `OUTRES`控制输出频率
   - 只保存必要时间点的结果

下一部分将讲解后处理与熔池跟踪分析。