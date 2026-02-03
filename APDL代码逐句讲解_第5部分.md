# APDL 代码逐句讲解 - 第5部分：边界条件与表面单元

## 第481-520行：创建表面效应单元（SURF152）

```apdl
!-----------------------------------!
! create surface effect element     !
!-----------------------------------!
TYPE, 2     ! SURF152
ESURF
```
**功能**：创建表面效应单元
- `TYPE, 2`：使用 SURF152 单元类型
- `ESURF`：在已有网格表面生成表面效应单元

```apdl
NSEL, S, LOC, Z, Height
NSEL, R, LOC, X, -PL/2, PL/2
NSEL, R, LOC, Y, -PW/2, PW/2
ESEL, ALL
ESLN, S, 0
ESURF, 0
```
**功能**：在打印区顶部表面创建表面单元
- `NSEL, S, LOC, Z, Height`：选择 Z=30000μm（顶部）
- `NSEL, R`：进一步筛选在打印区 XY 范围 (-25000, 25000) 的节点
- `ESURF, 0`：基于选中节点创建 SURF152 单元

```apdl
ESEL, S, TYPE, , 2
EMODIF, ALL, REAL, 2
```
**功能**：设置 SURF152 单元使用实常数2号
- `REAL, 2`：定义对流边界条件的参数组合

```apdl
NSEL, S, LOC, X, -Sub_length/2 
ESLN, S, 1
ESURF
NSEL, S, LOC, X, Sub_length/2 
ESLN, S, 1
ESURF
NSEL, S, LOC, Y, -Sub_width/2 
ESLN, S, 1
ESURF
NSEL, S, LOC, Y, Sub_width/2 
ESLN, S, 1
ESURF
```
**功能**：在基板四个侧面创建表面单元
- 选择 X=±70000μm（左右）和 Y=±50000μm（前后）位置的面

```apdl
NSEL, S, LOC, Z, Sub_height 
ESLN, S, 1
ESURF
ESEL, S, TYPE, , 2
EMODIF, ALL, REAL, 3
```
**功能**：在基板底部（Z=-30000μm）创建表面单元
- `REAL, 3`：特殊底部边界条件设置（恒定温度）

```apdl
ALLSEL, ALL
ESEL, S, TYPE, , 2
NSLE
NSEL, INVE
ESEL, INVE
```
**功能**：检查并反选非表面单元节点

---

## 第521-560行：定义边界条件

```apdl
!-----------------------------------!
! define boundary condition (BC)    !
!-----------------------------------!
NSEL, S, LOC, Z, Sub_height 
D, ALL, TEMP, TBottom_,,,
```
**功能**：在基板底面（Z=-30000μm）应用固定温度边界
- `D`：自由度约束命令
- `TEMP, TBottom_`：约束温度在 25°C
- **物理意义**：假设基板与散热片接触，温度恒定

```apdl
ALLSEL, ALL
NSEL, S, TYPE, , 2
SF, ALL, CONV, HF_, TS_
```
**功能**：在表面效应单元应用对流边界条件
- `SF`：表面载荷命令
- `CONV`：对流换热
- `HF_` = 20 W/m²·K：对流换热系数
- `TS_` = 25°C：环境温度

```apdl
*IF, Emissit_, EQ, 1, then
    SF, ALL, RDSF, Emissit_,,
*ENDIF
```
**功能**：条件应用辐射边界条件
- `RDSF`：辐射表面标志
- `Emissit_` = 0.8：辐射发射率
- **注意**：当前设置为启用辐射（通过 KEYOPT 提前配置）

```apdl
FINISH
```
**功能**：退出前处理器

---

## 第561-600行：定义实常数和初始温度

```apdl
! Define real constants !
R, 1,  , , , , , ,
R, 2,  , , , , , ,       
RMODIF, 2, 14, HF_
RMODIF, 2, 15, TBottom_
```
**功能**：定义两类实常数
- `R, 1`：通用实常数（备用）
- `R, 2`：对流边界实常数
- `RMODIF, 2, 14, HF_`：在14位置设置对流系数
- `RMODIF, 2, 15, TBottom_`：在15位置设置初始温度

```apdl
! Define initial temperature !
IC, ALL, TEMP, TBottom_
```
**功能**：设置所有节点初始温度为 25°C
- `IC`：初始条件命令
- **物理意义**：假设整个模型初始状态为室温

```apdl
TUNIF, TBottom_
```
**功能**：统一节点温度（确保初始均匀）

```apdl
! Define element birth parameters !
*DIM, ELEMENT_, ARRAY, n_mul_layer_
*DO, i, 1, n_mul_layer_, 1
    ELEMENT_(i) = 0
*ENDDO
```
**功能**：创建元素"出生"控制数组
- `*DIM, ELEMENT_`：定义数组
- `n_mul_layer_ = 60`：共60层
- **用途**：追踪每层元素的激活状态

```apdl
! Layer material change parameters!
*DIM, MATERIAL_CHANGE_TIME, ARRAY, n_mul_layer_
*DO, i, 1, n_mul_layer_, 1
    MATERIAL_CHANGE_TIME(i) = Layer_start_Time_%i% + t_heating_/2
*ENDDO
```
**功能**：定义材料转变时间数组
- 每层材料转变时间 = 层开始时间 + 加热时间的一半
- **物理意义**：在激光扫描到层中点时粉末变为致密材料

---

## 第601-640行：定义热源加载方式

```apdl
!-----------------------------------!
! Define heat source application    !
!-----------------------------------!
*DIM, Q_VOLU, TABLE, 7, 3, 1, TIME, TEMP
```
**功能**：定义表格数组（TABLE）描述热源参数
- `Q_VOLU`：体热源
- 维度：7×3×1
- X轴：时间（Time变量）
- Y轴：温度（Temp变量）

```apdl
Q_VOLU(0,0,1) = 0.0, -999, -999, -999, Q_AVE, 99, 0.0
Q_VOLU(0,1,1) = 0.0, 0.0, 250, 
...
Q_VOLU(0,6,1) = 0.0, 0.0, 3000, 
Q_VOLU(0,7,1) = 0.0, 0.0, 3000, 
```
**功能**：设置表格参数
- Y轴从0到7：对应不同温度区间
- 热源强度从0.0到Q_AVE

```apdl
! Define element birth and death control
*DO, i_layer, 1, n_mul_layer_, 1
    ! Select elements in this layer
    ESEL, S, MAT, , 1
    NSLE
    NSEL, R, LOC, Z, Activation_height_%i_layer-1%, Activation_height_%i_layer%
    ESEL, R
    CM, E_GRA%i_layer%, ELEM
    *SET, ELEMENT_(i_layer), 1
*ENDDO
```
**功能**：创建每层的元素组（用于生死单元控制）
- 按材料1（粉末）筛选
- 按Z坐标范围筛选（当前层的厚度）
- `CM`：组件创建命令（名为 E_GRA1, E_GRA2, ...）
- `ELEMENT_(i_layer)=1`：标记该组元素为待激活

```apdl
! Reset selection !
ALLSEL, ALL
```
**功能**：恢复全选

---

**第5部分讲解完毕**，涵盖了：
1. 表面效应单元（SURF152）的创建
2. 边界条件设置（对流、辐射、固定温度）
3. 实常数定义
4. 初始温度设置
5. 生死单元控制数组
6. 热源加载表格定义
7. 分层元素组创建（热源激活区）

**关键理解**：
1. **表面单元策略**：
   - 打印件顶部：SURF152单元（用于对流传热）
   - 基板侧边：SURF152单元（对流和辐射）
   - 基板底面：固定温度约束

2. **生死单元控制**：
   - 通过组件（Component）管理各层元素
   - ELEMENT_数组跟踪激活状态（0未激活，1激活）
   - MATERIAL_CHANGE_TIME数组控制粉末→致密材料转变时间

3. **热源加载**：
   - 使用TABLE数组定义温度/时间相关热源
   - 热源强度从Q_AVE开始随温度和位置变化
   - 实际加载将在求解阶段完成

下一部分将进入求解器设置（SOLVER），讲解热分析求解过程。