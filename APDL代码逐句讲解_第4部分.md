# APDL 代码逐句讲解 - 第4部分：基板材料与几何建模

## 第281-350行：定义 MAT 3（基板材料）

```apdl
!-------Substrate material-------!
! === 316L full dense thermal properties ====!
MAT, 3
```
**功能**：激活材料3（基板材料）
- 基板通常由相同材料制成，但可能有不同的物理状态

```apdl
! Thermal Conductivity, 1e+6, w/m*k = Kg*m/[s**3*K]
MP,   KXX,  3,  1e+6
```
**功能**：定义基板材料的初始热导率

```apdl
! Defines the temperature in celsius.
MPTEMP,,,,,,,,
MPTEMP, 01, 24.9
...
MPTEMP, 18, 1500
MPTEMP, 19, 2000
MPTEMP, 20, 3000
```
**功能**：定义20个温度点（减少到20个）
- **重点温度点**：2000°C（熔化）和3000°C（高温）

```apdl
! Enthalpy,(Ht-H25), J*m(-3)  
MPDE,   ENTH,3
MPDATA, ENTH,3,,0
MPDATA, ENTH,3,,270.25
...
MPDATA, ENTH,3,,13416.82
```
**功能**：定义基板材料的焓值
- **注意**：高于600°C后与MAT 2相同
- 基板在打印过程中温度较低

```apdl
! Thermal Conductivity, 1e+6, w/m*k = Kg*m/[s**3*K]
MPDE,   KXX,3
MPDATA, KXX,3,, 15.60E6
...
MPDATA, KXX,3,, 54.29E6*KXX_ENHNC
```
**功能**：定义基板材料的 X 方向热导率
- **与MAT 2相同**：基板材料与打印件相同（316L）
- **假设**：基板是致密的

```apdl
MPDE,   KYY,3
...
MPDE,   KZZ,3
MPDATA, kZZ,3,, 15.60E6
...
MPDATA, KZZ,3,, 38.74E6*KZZ_ENHNC_T5
MPDATA, KZZ,3,, 54.29E6*KZZ_ENHNC
```
**功能**：定义Y和Z方向热导率
- **特点**：各向同性（三个方向相同）

---

## 第351-370行：几何建模准备

```apdl
!-----------------------!
! preprocess - modeling !
!-----------------------!
CSYS, 0
```
**功能**：设置坐标系为笛卡尔坐标系（直角坐标系）

```apdl
/PSYMB, LDIR, 1
/PSYMB, ADIR, 1
```
**功能**：激活显示线方向（Line Direction）和面方向（Area Direction）

```apdl
/PNUM, KP, 1
/PNUM, LINE, 1
/PNUM, AREA, 1
/PNUM, VOLU, 1
```
**功能**：激活显示关键点、线、面和体积的编号

```apdl
SELTOL, 1e-10
```
**功能**：设置选择容差（Selection Tolerance）为1×10⁻¹⁰
- **用途**：精确选择模型中的几何实体

---

## 第371-379行：创建关键点（KP）

```apdl
!Create KPs
K, 1, -Sub_length/2, -Sub_width/2, -10000,
```
**功能**：创建关键点1
- **坐标**：
  - X = -35000 μm（基板长度一半）
  - Y = -25000 μm（基板宽度一半）
  - Z = -10000 μm（基底下方）

```apdl
K, 2, Sub_length/2, -Sub_width/2, -10000,
K, 3, -Sub_length/2, Sub_width/2, -10000,
K, 4, Sub_length/2, Sub_width/2, -10000,
```
**功能**：创建关键点2-4（基板底部四个角点）

```apdl
K, 5, -PL/2, -PW/2, -10000,
K, 6, PL/2, -PW/2, -10000,
K, 7, -PL/2, PW/2, -10000,
K, 8, PL/2, PW/2, -10000,
```
**功能**：创建关键点5-8（打印区域的四个角点）

```apdl
K, 9, -Sub_length/2, -Sub_width/2, -20000,
...
K, 15, -Sub_length/2, -Sub_width/2, Height
```
**功能**：创建关键点9-15
- **关键点9-12**：在 Z = -20000 μm 的基板底部
- **关键点13-15**：Z坐标从基底到打印件高度
- **Height = 30000 μm**：打印件顶部高度

---

## 第380-400行：创建线（Lines）和设置网格划分数

```apdl
! Create lines !
LSTR,         1,         2,     !L1
LSTR,         2,         4,     !*L2
LSTR,         4,         3,     !L3
LSTR,         3,         1,     !L4
```
**功能**：创建四边形基板的四条边

```apdl
! specify the NDIV and SR of lines!
LESIZE, 1, , , PL/rb, 1, 
LESIZE, 2, , , 4, 1, 
LESIZE, 3, , , PL/rb, 1,
LESIZE, 4, , , 4, 1, 
```
**功能**：设置线的网格划分数和间距比
- **L1**：划分数 = PL/rb = 50000/500 = 100
- **L2**：划分数 = 4（固定）
- **SR=1**：均匀划分

```apdl
! lines for laser scaned zone !
LSTR,         5,         6,     !L5
LSTR,         6,         8,     !*L6
LSTR,         8,         7,     !L7
LSTR,         7,         5,     !L8
```
**功能**：创建打印区域的四条边

```apdl
LESIZE, 5, , , PL/rb, 1,
LESIZE, 6, , , 4, 1, 
LESIZE, 7, , ,PL/rb , 1,
LESIZE, 8, , , 4, 1, 
```
**功能**：设置打印区边缘线的网格划分数

```apdl
LSTR,         5,         1,     !L9
...
LSTR,         7,         3,     !L12
```
**功能**：连接打印区和基板边缘的线

```apdl
LESIZE, 9, , , NDIV_XY, SR_XY,
...
LESIZE, 12, , , NDIV_XY, SR_XY,
```
**功能**：设置这些过渡线的网格划分数和间距比
- **NDIV_XY = 20**（每边）
- **SR_XY = 1.5**（从中心向边缘网格逐渐变大）

---

## 第401-440行：创建面（Areas）、体（Volumes）和网格划分

```apdl
! Create areas !
AL, 1, 10, 5, 9,            !A1
...
AL, 5, 6, 7, 8,             !A5
```
**功能**：创建五个面，拼成完整的基板区域

```apdl
AGLUE, 1, 2, 3, 4, 5,
```
**功能**：胶合（Glue）所有面，确保几何体连接

```apdl
! lines for convex plate side !
LSTR,         5,         9,     !L13
...
LSTR,         7,         11,    !L16
```
**功能**：创建垂直侧面的线

```apdl
LESIZE, 13, , , NDIV_Vol, SR_Vol,
...
LESIZE, 16, , , NDIV_Vol, SR_Vol,
```
**功能**：设置侧面线的网格划分

```apdl
! lines of convex plate heigth !
...
! lines for convex plate bottom !
...
```
**功能**：创建底部线和设置网格（类似）

```apdl
! Create volumes !
V, 10, 12, 11, 9, 6, 8, 7, 5,       !V1
...
V, 11, 11, 12, 12, 3, 7, 8, 4,      !V5
```
**功能**：创建打印件主体体积（共5个体积）

```apdl
! create convex plate !
VDRAG, 6, , , , , , 26,             !V6
VDRAG, 1, 2, 3, 4, 5, , 25,          !V7-V10
VDRAG, 40, , , , , ,27,           !V11
```
**功能**：使用拖动（VDRAG）创建其他体积

```apdl
VSEL, ALL
VGLUE, ALL
ALLSEL, ALL
```
**功能**：选择所有体并胶合（确保连续网格）

---

## 第441-480行：网格划分

```apdl
! specify the element !
TYPE, 1 
MAT, 3
```
**功能**：设置默认单元类型和材料为基板（SOLID70单元）

```apdl
! specify the element shape !          
MSHAPE, 0, 3D
```
**功能**：指定网格形状为六面体

```apdl
!  set the element size of triangle face on pentahedron  !
*Do, N_Volu, 2, 5, 1
     *if, N_Volu, NE, 5, THEN,
          ...（局部网格尺寸设置）
     *ENDIF
*ENDDO
```
**功能**：循环设置五棱柱面上的网格尺寸

```apdl
! mesh pentahedron !
*Do, N_Volu, 2, 5, 1
     ALLSEL, ALL
     VSWEEP, N_Volu,
*ENDDO
```
**功能**：扫掠网格划分五棱柱区域

```apdl
! **warning**mesh the other part of substrate!
ALLSEL, ALL
TYPE, 1
MAT, 3
! **err**Mapped mesh ! 
MSHKEY, 1   
MSHAPE, 0, 3D
VMESH, 1,
VMESH, 6, 11, 1
```
**功能**：映射网格划分基板区域
- `MSHKEY, 1`：强制使用映射网格（要求几何结构化）

```apdl
! mesh powder bed !
ALLSEL, ALL
TYPE, 1
MAT, 1
MSHKEY, 1
MSHAPE, 0, 3D
VMESH, 11,12,1
```
**功能**：使用MAT 1（粉末）划分打印件区域的网格

```apdl
ALLSEL,ALL
NUMMRG,all,,,,,
NUMCMP,all
ALLSEL, ALL 
```
**功能**：合并并压缩所有实体编号（清理重复项）

---

**第4部分讲解完毕**，涵盖了：
1. 基板材料（MAT 3）的热物性
2. 几何建模关键点创建
3. 线和面的创建
4. 体积建模技巧
5. 结构化网格划分策略
6. 材料分区域分配（基板/打印区/粉末区）

**关键理解**：
1. **分层材料分配**：
   - 基板区域：MAT 3
   - 打印区域：默认MAT 1（粉末）
   - 后续可通过"生死单元"激活为MAT 2（致密）
   
2. **网格划分策略**：
   - 底部基板使用映射网格（六面体网格）
   - 中间过渡层使用扫掠网格
   - 表面单元将在后续添加

3. **分层几何结构**：
   - 基板：Sub_height = -30000μm
   - 打印区：高度0-Height(30000μm)
   - 整个模型高度范围：-30000μm至+30000μm

下一部分将讲解边界条件设置和热处理模型。