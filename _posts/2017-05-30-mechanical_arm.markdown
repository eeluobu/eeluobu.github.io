---
layout:     post
title:      "多自由度机械臂控制"
subtitle:   "——BP神经网络在运动学逆解中的应用"
date:       2017-05-30
author:     "luobu"
header-img: "img/post.jpg"
catalog:    true
tags:
    - 机械臂
    - 神经网络
    - 运动学
    - matlab
---


-----

机械臂的控制，在进行运动学分析时，需要计算的主要就是两个部分，分别是机械臂运动学正解和运动学逆解。

### 运动学分析
采用D-H法来进行机械臂空间的建模和运动学求解，D-H坐标建立法在1955年由Denavit和Hartenberg提出，由于在实际应用中简单实用，同时方便机械臂的建模和解算，在机械臂运动学研究中拥有广泛的应用。

#### 机械臂坐标系
采用机器人导论上的方法，建立我使用的5自由度机械臂的坐标系。   

> 1、找到各关节轴并标出这些轴线的延长线。在接下来的步骤中，只考虑两个相邻的关节轴 i 和 i+1；  
2、找出关节轴 i 和 i+1 之间的公垂线，以关节轴 i 和 i+1 的公垂线与关节轴 i 的交点作为连杆坐标系 i 的原点；  
3、规定 Z<sub>i</sub> 轴沿关节轴 i 的指向；  
4、规定 X<sub>i</sub> 轴方向为公垂线的指向，如果关节轴 i 和 i+1 相交，则规定 X<sub>i</sub> 轴垂直于关节轴 i 和 i+1 所在的平面；  
5、按照左手定则确定 Y<sub>i</sub> 轴；  
6、当第一个关节变量为 0 时，规定坐标系 {0} 和 {1} 重合。对于坐标系 {N}，其原点和 X<sub>n</sub> 的方向可以任意选取。

下图是根据所采用的5自由度机械臂的结构，画出的相应的机械臂连杆坐标系：
![img](/img/post/mechanical/zbx.jpg)

各个杆件之间的 D-H 参数定义：

> a<sub>i</sub>：沿着 X<sub>i-1</sub> 轴，将 Z<sub>i-1</sub> 移动到 Z<sub>i</sub> 的距离；  
α<sub>i</sub>：沿着 X<sub>i-1</sub> 轴，将 Z<sub>i-1</sub> 旋转到 Z<sub>i</sub> 的角度；  
d<sub>i</sub>：沿着 Z<sub>i-1</sub> 轴，将 X<sub>i-1</sub> 移动到 X<sub>i</sub> 的距离；  
θ<sub>i</sub>：沿着 Z<sub>i-1</sub> 轴，将 X<sub>i-1</sub> 旋转到 X<sub>i</sub> 的角度。

经过测量，下面是我毕设所使用的5自由度舵机机械臂的 D-H 参数表：
![img](/img/post/mechanical/dh.jpg)

#### 正运动学分析

有了 D-H 参数后，就可以使用 matlab 将所使用的机械臂仿真出来了，下面是建立各个连杆的代码，其中 SerialLink 函数为 robotics 工具箱的建立机械臂连杆连接函数：

``` matlab
L1 = Link('d', 0, 'a', 0, 'alpha', pi/2);
L2 = Link('d', 0, 'a', 10.4, 'alpha', 0);
L3 = Link('d', 0, 'a', 10, 'alpha', 0);
L4 = Link('d', 0, 'a', 2.2, 'alpha', -pi/2);
L5 = Link('d', 12, 'a', 0, 'alpha', 0);
%使用 SerialLink 函数建立机械臂各连杆关系
bot = SerialLink([L1 L2 L3 L4 L5]);

%使用 teach 函数画出机械臂仿真模型
bot.teach;
```

通过 robotics 工具的 fkine 和 ikine 函数，可以很方便的进行机械臂的正、逆解算。fkine 函数为机械臂正解函数，原理即为各个连杆齐次矩阵依次相乘，如下两行代码是等效的。

``` matlab
%robotics工具箱的正解函数fkine
%θ1~θ5为各个自由度变化角度
T = bot.fkine([θ1 θ2 θ3 θ4 θ5]);

%T1~T5分别为各自由度齐次坐标矩阵
%T1~T5可以由各θ角得出
T = T1*T2*T3*T4*T5;
```

#### 逆运动学分析

matlab 中的 ikine 函数采用传统矩阵逆乘解法，这种求解方法的缺点：  

> 1、当机械臂各关节角度间存在复杂的耦合关系，在运动学逆解时解耦的过程会很复杂；  
2、机械臂末端执行器的空间位置和各关节角度与是一对多的关系，在解析时很有可能得到多个解，此时还要根据机构的结构特点选取较优解。

为了避免这些问题出现，采用BP神经网络作为机械臂运动学逆解的算法，训练出来的模型可以避免传统解析法的不足，不仅可以提升运算效率，还能提高控制的响应速度。

### BP神经网络
BP神经网络是一种多层前馈网络，BP即Back Propagation反向传播，按误差逆传播算法来进行训练。典型的 BP 神经网络拓扑结构有：输入层（input）、隐层（hide layer）和输出层（output layer）。每层之间采用全互联方式，同层之间则不存在连接。下面是一个典型的三层BP神经网络拓扑结构：
![img](/img/post/mechanical/bp.jpg)

设计了一个单层隐含层的神经网络模型，作为机械臂逆运动学的求解模型。根据最终描述位姿的齐次矩阵的特点，设计输入层和输出层的神经元节点数。  
将输入层设计为12个神经元节点，分别用于输入正运动学方程矩阵公式中，描述机械臂末端空间位姿的12个变量。分别对应齐次变换矩阵中的相应元素，如第一个神经元输入对应矩阵T的(1, 1)元素，第二个对应(2, 1)，依次类推。下面是输入向量：

``` matlab
%其中nx表示T(1, 1)，其他依次类推，如：pz为T(3, 4)
x = [nx, ny, nz, ox, oy, oz, ax, ay, az, px, py, pz];
```

将输出层设计为5个神经元节点，用来表示5个自由度关节的旋转角度θ1~θ5，下面是输出向量：

``` matlab
o = [θ1, θ2, θ3, θ4, θ5];
```

#### 训练数据

产生合适的训练样本数据对训练神经网络来说非常重要。  
结合5自由度机械臂正运动学模型，以及各个自由度关节变量 θi 的取值范围，将θi数据等分为 100 组，作为神经网络训练的输出数据部分。数据范围如下：  
![img](/img/post/mechanical/theta.jpg)

利用正运动学求得的末端执行机构坐标系变换矩阵 T，求出 θi 对应的 12 个位姿相关数据，作为神经网络训练的输入数据。  
下面是训练数据生成代码：

``` matlab
T=[];
%矩阵x为训练输入数据
x=[];
theta_t1 = (-pi)  :(pi/50)   :(pi);
theta_t2 = (-pi/2):(pi/100)  :(pi/2);
theta_t3 = (-pi/2):(pi/100)  :(pi/2);
theta_t4 = (-pi/2):(pi*3/200):(pi);
theta_t5 = (-pi)  :(pi/50)   :(pi);
alpha1=pi/2;
a1=0;
d1=0;
alpha2=0;
a2=10.4;
d2=0;
alpha3=0;
a3=10;
d3=0;
alpha4=-pi/2;
a4=2.2;
d4=0;
alpha5=0;
a5=0;
d5=12;
for i= 1:100
    theta1 = theta_t1(i);
    theta2 = theta_t2(i);
    theta3 = theta_t3(i);
    theta4 = theta_t4(i);
    theta5 = theta_t5(i);
T1=[cos(theta1),-sin(theta1)*cos(alpha1),sin(theta1)*sin(alpha1),a1*cos(theta1);...
    sin(theta1),cos(theta1)*cos(alpha1),-cos(theta1)*sin(alpha1),a1*sin(theta1);...
    0,sin(alpha1),cos(alpha1),d1;...
    0,0,0,1];
T2=[cos(theta2),-sin(theta2)*cos(alpha2),sin(theta2)*sin(alpha2),a2*cos(theta2);...
    sin(theta2),cos(theta2)*cos(alpha2),-cos(theta2)*sin(alpha2),a2*sin(theta2);...
    0,sin(alpha2),cos(alpha2),d2;...
    0,0,0,1];
T3=[cos(theta3),-sin(theta3)*cos(alpha3),sin(theta3)*sin(alpha3),a3*cos(theta3);...
    sin(theta3),cos(theta3)*cos(alpha3),-cos(theta3)*sin(alpha3),a3*sin(theta3);...
    0,sin(alpha3),cos(alpha3),d3;...
    0,0,0,1];
T4=[cos(theta4),-sin(theta4)*cos(alpha4),sin(theta4)*sin(alpha4),a4*cos(theta4);...
    sin(theta4),cos(theta4)*cos(alpha4),-cos(theta4)*sin(alpha4),a4*sin(theta4);...
    0,sin(alpha4),cos(alpha4),d4;...
    0,0,0,1];
T5=[cos(theta5),-sin(theta5)*cos(alpha5),sin(theta5)*sin(alpha5),a1*cos(theta5);...
    sin(theta5),cos(theta5)*cos(alpha5),-cos(theta5)*sin(alpha5),a1*sin(theta5);...
    0,sin(alpha5),cos(alpha5),d5;...
    0,0,0,1];

T=[T,T1*T2*T3*T4*T5];
end
n=T(1:3, 1:4:400);
o=T(1:3, 2:4:400);
a=T(1:3, 3:4:400);
p=T(1:3, 4:4:400);
for j= 1:100
    x=[x,[n(1,j), n(2,j), n(3,j), o(1,j), o(2,j), o(3,j), a(1,j), a(2,j), a(3,j), p(1,j), p(2,j), p(3,j)]'];
end
%矩阵theta为训练输出数据
theta =[theta_t1(:,1:100);theta_t2(:,1:100);theta_t3(:,1:100);theta_t4(:,1:100);theta_t5(:,1:100)];

train_input = roundn(x,-4);
train_theta = roundn(theta,-4);
save 'train_input' train_input;
save 'train_theta' train_theta;
```

#### 训练参数
处理和生成完训练数据后，将70%的数据作为训练数据（Training），15%作为验证数据（Validation），15%作为测试数据（Testing）。设置网络的 net.divideFcn 参数为 dividerand，即将 100 组数据按照设定好的比例，随机分配这三种数据。最大训练次数 epochs 设置为1000，训练目标最小误差 goal 设置为0.0001。下面是神经网络训练程序：

``` matlab
load train_input;
load train_theta;

x = train_input;
t = train_theta;

% Levenberg-Marquardt backpropagation.
trainFcn = 'trainlm';  

% Create a Fitting Network
hiddenLayerSize = 10;
net = fitnet(hiddenLayerSize,trainFcn);

net.divideFcn = 'dividerand';  % Divide data randomly
net.divideMode = 'sample';  % Divide up every sample
net.divideParam.trainRatio = 70/100;
net.divideParam.valRatio = 15/100;
net.divideParam.testRatio = 15/100;
net.trainParam.epochs = 1000;
net.trainParam.goal = 0.0001;

% Train the Network
[net,tr,y,e] = train(net,x,t);

% Test the Network
performance = perform(net,t,y)

% View the Network
view(net)

% Generate a matrix-only MATLAB function for neural network code
% generation with MATLAB Coder tools.
genFunction(net,'myNeuralNetworkFunction','MatrixOnly','yes');
y = myNeuralNetworkFunction(x);
```

下图是LM方法对此BP神经网络进行训练时的网络训练误差曲线：
![img](/img/post/mechanical/epochs.jpg)
以关节转角变量θ1为例，通过BP模型输出的结果与测试数据结果的对比：
![img](/img/post/mechanical/error.jpg)

#### 生成预测模型
使用 matlab 的 genFunction 函数，生成训练好的网络模型函数，作为机械臂逆运动学解算的预测函数。输入为末端机械手的旋转平移变换齐次矩阵，输出为各个自由度关节角度。

``` matlab
function [y1] = myNeuralNetworkFunction(x1)
%  初始化各层的权值和阀值部分
%  Input 1
%  x1_step1_xoffset
%  x1_step1_gain
%  x1_step1_ymin
%
%  Layer 1
%  b1
%  IW1_1
%
%  Layer 2
%  b2
%  LW2_1
%
%  Output 1
%  y1_step1_xoffset
%  y1_step1_gain
%  y1_step1_ymin
%  初始化结束

  % ===== SIMULATION ========
  
  % Dimensions
  Q = size(x1,2); % samples
  
  % Input 1
  xp1 = mapminmax_apply(x1,x1_step1_gain,x1_step1_xoffset,x1_step1_ymin);
  
  % Layer 1
  a1 = tansig_apply(repmat(b1,1,Q) + IW1_1*xp1);
  
  % Layer 2
  a2 = repmat(b2,1,Q) + LW2_1*a1;
  
  % Output 1
  y1 = mapminmax_reverse(a2,y1_step1_gain,y1_step1_xoffset,y1_step1_ymin);
end

% ===== MODULE FUNCTIONS ========

% Map Minimum and Maximum Input Processing Function
function y = mapminmax_apply(x,settings_gain,settings_xoffset,settings_ymin)
  y = bsxfun(@minus,x,settings_xoffset);
  y = bsxfun(@times,y,settings_gain);
  y = bsxfun(@plus,y,settings_ymin);
end

% Sigmoid Symmetric Transfer Function
function a = tansig_apply(n)
  a = 2 ./ (1 + exp(-2*n)) - 1;
end

% Map Minimum and Maximum Output Reverse-Processing Function
function x = mapminmax_reverse(y,settings_gain,settings_xoffset,settings_ymin)
  x = bsxfun(@minus,y,settings_ymin);
  x = bsxfun(@rdivide,x,settings_gain);
  x = bsxfun(@plus,x,settings_xoffset);
end
```

如果想要将这个模型移植到其他不能运行 matlab 代码的平台（如 arm 架构的板子），来实现对机械臂的控制，可以通过matlab coder工具将上面的函数生成 c 代码函数，即可实现跨平台运行。