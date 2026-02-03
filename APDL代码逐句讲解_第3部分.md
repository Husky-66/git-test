# APDL 代码逐句讲解 - 第3部分：前处理与材料属性定义

## 第104-113行：进入前处理器并定义单元类型

```apdl
!--------------------------------------------------!
! preprocess - define material property & modeling !
!--------------------------------------------------!
```
**功能**：注释分隔符

```apdl
/PREP7
```
**功能**：进入前处理器（Preprocessor）
- ANSYS 的前处理模块，用于建模、定义材料、划分网格等

```apdl
/UNITS, umks,,,,,,,
```
**功能**：设置单位系统
- `umks`：微米-千克-秒单位系统
  - 长度：μm（微米）
  - 质量：kg（千克）
  - 时间：s（秒）
  - 温度：°C（摄氏度）
- **重要**：所有后续输入都要符合此单位系统

```apdl
ET, 1,  SOLID70
```
**功能**：定义单元类型1为 SOLID70
- `ET`：Element Type（单元类型）命令
- `1`：单元类型编号
- `SOLID70`：3D 热分析实体单元（8节点六面体）
- **用途**：温度场计算

```apdl
ET, 2,  SURF152,
```
**功能**：定义单元类型2为 SURF152
- `SURF152`：3D 表面效应单元
- **用途**：施加对流和辐射边界条件

```apdl
KEYOPT, 2,  4,  1
```
**功能**：设置 SURF152 的关键选项4为1
- `KEYOPT(4) = 1`：包含对流换热

```apdl
KEYOPT, 2,  5,  0
```
**功能**：设置 SURF152 的关键选项5为0
- `KEYOPT(5) = 0`：不包含辐射（将通过综合换热系数考虑）

```apdl
KEYOPT, 2,  8,  2
```
**功能**：设置 SURF152 的关键选项8为2
- `KEYOPT(8) = 2`：基于基础单元的几何形状

```apdl
KEYOPT, 2,  9,  0
```
**功能**：设置 SURF152 的关键选项9为0
- `KEYOPT(9) = 0`：默认设置

---

## 第114-130行：粉末到致密材料转换参数

```apdl
!!!=============!!!
!!! powder2bulk !!!
!!!=============!!!
```
**功能**：注释，标识粉末到致密材料转换部分

```apdl
*SET, TEMP_M1TOM2, (1350 + 0)   ! TEMP when MAT1 changes to MAT2
```
**功能**：定义材料转换温度
- `TEMP_M1TOM2 = 1350 °C`
- 当温度超过此值时，粉末（MAT 1）转换为致密材料（MAT 2）
- 接近316L不锈钢的熔点（约1400°C）

```apdl
! Enhancement factors for thermal conductivity
*SET, KXX_ENHNC_, 5.0
```
**功能**：定义 X 方向热导率增强因子
- `KXX_ENHNC_ = 5.0`
- 在高温下，粉末热导率增强（模拟熔化效应）

```apdl
*SET, KYY_ENHNC_, 5.0
```
**功能**：定义 Y 方向热导率增强因子
- `KYY_ENHNC_ = 5.0`

```apdl
*SET, KZZ_ENHNC_, 15.0
```
**功能**：定义 Z 方向热导率增强因子
- `KZZ_ENHNC_ = 15.0`
- **注意**：Z 方向增强更多（15倍 vs 5倍）
- **物理意义**：模拟熔池深度方向的热传导增强

```apdl
! Define temperature interpolation range
*SET, T_MIN, 1500
```
**功能**：定义温度插值范围的最小值
- `T_MIN = 1500 °C`

```apdl
*SET, T_MAX, 3000
```
**功能**：定义温度插值范围的最大值
- `T_MAX = 3000 °C`

```apdl
! Define temperature points  
*SET, T1, 1600
*SET, T2, 1700
*SET, T3, 1800
*SET, T4, 1900
*SET, T5, 2000
```
**功能**：定义5个特定温度点
- 用于在不同温度下设置不同的增强因子

```apdl
! Compute enhancement factors at specific temperatures  
*SET, KXX_ENHNC_T1, 3
*SET, KYY_ENHNC_T1, 3
```
**功能**：定义在温度 T1 时的 X、Y 方向增强因子
- `KXX_ENHNC_T1 = 3`
- `KYY_ENHNC_T1 = 3`

```apdl
*SET, KZZ_ENHNC_T1, 5
*SET, KZZ_ENHNC_T2, 5
*SET, KZZ_ENHNC_T3, 5
*SET, KZZ_ENHNC_T4, 5
*SET, KZZ_ENHNC_T5, 5
```
**功能**：定义在不同温度下 Z 方向的增强因子
- 所有温度点都使用 `5` 作为增强因子

---

## 第131-135行：温度场计算说明

```apdl
!!!===============================!!!
!!! Temperature field calculation !!!
!!!===============================!!!
```
**功能**：注释分隔符

```apdl
!---------------------------------!
! preprocess - material properity !
!---------------------------------!
```
**功能**：注释分隔符

---

## 第136-200行：定义 MAT 1（316L 粉末材料）

```apdl
!------316L POWDER material------!
MAT, 1
```
**功能**：激活材料1（316L 粉末）
- `MAT, 1`：后续材料属性定义都针对材料1

```apdl
! 316L , 1e+6, w/m*k = Kg*m/[s**3*K]
MP, KXX, 1, 1e+6
```
**功能**：定义材料1的 X 方向热导率初始值
- `MP`：Material Property（材料属性）命令
- `KXX`：X 方向热导率
- `1e+6`：初始值（将被后续温度相关数据覆盖）
- **单位说明**：在 μm-kg-s 系统中，热导率单位为 kg·μm/(s³·K)

```apdl
! Defines the temperature in celsius.
MPTEMP,,,,,,,,
MPTEMP, 01, 24.9
MPTEMP, 02, 100
MPTEMP, 03, 200
...
MPTEMP, 23, 3000
```
**功能**：定义23个温度点
- `MPTEMP`：定义材料属性的温度表
- 温度范围：24.9°C 到 3000°C
- **关键温度点**：
  - 1358°C：固相线温度
  - 1392°C：液相线温度
  - 1500°C 以上：液态

```apdl
! Enthalpy,(Ht-H25), J*m(-3)       
MPDE,   ENTH,1
```
**功能**：定义材料1的焓值（Enthalpy）
- `MPDE`：Material Property Data Entry（材料属性数据输入）
- `ENTH`：焓值
- **单位**：J/m³ = kg/(μm·s²)（在 μm-kg-s 系统中）

```apdl
MPDATA, ENTH,1,,0
MPDATA, ENTH,1,,108.1
MPDATA, ENTH,1,,260.01
...
MPDATA, ENTH,1,,13416.82
```
**功能**：输入23个温度点对应的焓值
- 从 0（25°C）到 13416.82（3000°C）
- **物理意义**：包含显热和相变潜热
- **注意**：在1358-1392°C之间有大幅跳跃（相变潜热）
  - 2423.784 → 7836.45（增加约5412 J/m³）

```apdl
! Thermal Conductivity, 1e+6, w/m*k = Kg*m/[s**3*K]
MPDE,   KXX,1
MPDATA, KXX,1,, 0.23042E6
MPDATA, KXX,1,, 0.29343E6
...
MPDATA, KXX,1,, 54.29E6*KXX_ENHNC
```
**功能**：定义 X 方向热导率随温度变化
- **低温区**（24.9-1392°C）：0.23-1.71 E6
- **高温区**（1500-2000°C）：应用增强因子
  - 例如：`31.91487E6 * KXX_ENHNC_T1 = 31.91487E6 * 3`
- **最高温**（3000°C）：`54.29E6 * KXX_ENHNC_ = 54.29E6 * 5`

```apdl
MPDE,   KYY,1
MPDATA, KYY,1,, 0.23042E6
...
MPDATA, KYY,1,, 54.29E6*KYY_ENHNC
```
**功能**：定义 Y 方向热导率
- 与 X 方向相同（各向同性在 XY 平面）

```apdl
MPDE,   KZZ,1
MPDATA, KZZ,1,, 0.23042E6
...
MPDATA, KZZ,1,, 54.29E6*KZZ_ENHNC
```
**功能**：定义 Z 方向热导率
- **关键差异**：高温区使用 `KZZ_ENHNC_ = 15`（而非5）
- **物理意义**：Z 方向（深度方向）热传导更强

---

## 第201-280行：定义 MAT 2（316L 致密材料）

```apdl
! === 316L full dense thermal properties ====!
MAT, 2
```
**功能**：激活材料2（316L 致密材料）

```apdl
! Thermal Conductivity, 1e+6, w/m*k = Kg*m/[s**3*K]
MP,   KXX,  2,  1e+6
```
**功能**：定义材料2的初始热导率

```apdl
! Defines the temperature in celsius.
MPTEMP,,,,,,,,
MPTEMP, 01, 24.9
...
MPTEMP, 23, 3000
```
**功能**：定义23个温度点（与 MAT 1 相同）

```apdl
! Enthalpy,(Ht-H25), J*m(-3)  
MPDE,   ENTH,2
MPDATA, ENTH,2,,0
MPDATA, ENTH,2,,270.25
...
MPDATA, ENTH,2,,13416.82
```
**功能**：定义致密材料的焓值
- **差异**：致密材料的焓值更高（密度更大）
- 例如：100°C时，粉末108.1 vs 致密270.25

```apdl
! Thermal Conductivity, 1e+6, w/m*k = Kg*m/[s**3*K]
MPDE,   KXX,2
MPDATA, KXX,2,, 15.60E6
MPDATA, KXX,2,, 16.60E6
...
MPDATA, KXX,2,, 54.29E6*KXX_ENHNC
```
**功能**：定义致密材料的 X 方向热导率
- **关键差异**：低温区热导率更高
  - 粉末：0.23E6，致密：15.60E6（约67倍）
- **物理意义**：致密材料导热性能远优于粉末

```apdl
MPDE,   KYY,2
...
```
**功能**：定义 Y 方向热导率（与 X 相同）

```apdl
MPDE,   KZZ,2
MPDATA, KZZ,2,, 15.60E6
...
MPDATA, KZZ,2,, 31.91487E6*KZZ_ENHNC_T1
MPDATA, KZZ,2,, 34.22989E6*KZZ_ENHNC_T2
MPDATA, KZZ,2,, 35.84492E6*KZZ_ENHNC_T3
MPDATA, KZZ,2,, 37.45994E6*KZZ_ENHNC_T4
MPDATA, KZZ,2,, 38.74E6*KZZ_ENHNC_T5
MPDATA, KZZ,2,, 54.29E6*KZZ_ENHNC
```
**功能**：定义 Z 方向热导率
- **注意**：在不同温度点使用不同的增强因子
  - T1(1600°C)：`KZZ_ENHNC_T1 = 5`
  - T2-T5：也是5
  - 3000°C：`KZZ_ENHNC_ = 15`

---

**第3部分讲解完毕**，涵盖了：
1. 前处理器设置
2. 单元类型定义（SOLID70、SURF152）
3. 粉末到致密材料转换参数
4. MAT 1（粉末）的热物性
5. MAT 2（致密）的热物性

**关键理解**：
- **各向异性热导率**：Z 方向增强因子（15）> XY 方向（5）
- **相变潜热**：通过焓值跳跃体现（1358-1392°C）
- **粉末 vs 致密**：致密材料热导率约为粉末的67倍

下一部分将讲解基板材料（MAT 3）和几何建模。
