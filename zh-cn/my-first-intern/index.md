# 第一份实习


2021 年大二结束后的暑假，托关系找了份实习。一家医疗器械公司，主营体外检测仪器和配套试剂。安排我到他们做快速荧光 PCR 的项目里。说起来也蛮有趣，我一个臭机械本科生，六个月里竟然前后干了机械电子软件三种工作（大约各两个月）。流水账式地记录下几个主要内容。

## 软件原型

仪器准备投放到基层医院，所以需要一个比较易用的界面，方便操作员快速培训上岗。正好我入职啥也不懂，安排我设计下软件原型。咱也不是艺术生，配色啥的咱也不懂，主要就是设计 UI/UX 的操作流程，点这个按钮跳到什么什么界面这样。原型软件领导直接给买了两份 axure，不过似乎 figma 更火的样子。现在看来软件原型也确实是一个很好的入门项目，快速熟悉了基本的仪器原理和运行流程。

## 步进电机驱动调研

唉，下个学期才开电机的课，就当提前预习了吧。驱动器一般分为脉冲型和总线型。

- 脉冲型：

  你只需要发脉冲，一个脉冲走一步，全步半步微步什么的可以用其他引脚配置。缺点是运动加减速全部都要自己控制，实时性要求比较高，如果要求闭环控制则更复杂；胜在价格便宜，一片不过十几块钱

- 总线型：

  - 驱动模块：

    当时正在使用一款国产 CAN 总线驱动器，小毛病不断而且价格不菲，一台机器要 12 个，每个要 300 多元，成本上希望压榨一下。领导的意思是自己搞个 FPGA 接脉冲型芯片，12 个驱动器一起控制了。我想了想 FPGA 开发难度实在是有点大，而且只带 12 个是不是有点大材小用 ~~关键是项目里没人会~~

  - 驱动芯片：

    推荐了 [Trinamic](https://www.trinamic.com/) 的 `TMC5160`，一个是功能强效果好，另外公司里另一个项目组就选了同款芯片，有打好的电路板可以直接拿来用。芯片一片 80 左右，单片机用 F103 最低配都绰绰有余，也可以用个资源多的型号同时控多路 SPI 挂多个芯片。算上外围元件 + 制板费也比原来低不少，而且还不用自己开发

## 荧光镜片组外壳迭代

同类机器一般用激光器做光源，虽然效果超棒但价格也很美丽。我们这边打算拿 5W LED 灯珠 + 透镜 + 窄带滤光片，反正又不是成像，能用就行，中间还需要加一片二向色镜滤掉 LED 反射信号，保留荧光信号分给 [PD]^(Photon Detector)。第一版原型已经验证了可行性，只不过信号强度比较有限，希望还能提升一点。

按我的初中光学水平，聚焦嘛，几组透镜焦点对上就行。同时尽量缩短光传播的距离减少衰减。所以就需要焦距极短的透镜，聚光能力更强。不过我们仪器体积有限，上不了太大的镜片，只能用鼓形透镜，我们需求的型号又没有现货，第一版跟厂家定制把球形透镜削成鼓形，价格也是非常美丽。

领导提出让我试一下不严格对齐焦点，优先压缩衰减距离，3D 打印个模型试试看效果。同时因为不聚焦了嘛，平凸透镜换更便宜的非球面透镜，滤光片也换了国产供应商进一步压缩成本。SolidWorks 走起。利用 SW 的型腔功能，可以先画镜片的装配，再从一个基体里把镜片的部分切掉。改模型的过程中还发现上一版光路中心偏移了 0.5mm LOL.

因为探头尺寸小了好多，LED 电路板也得重新画，咱也不会 AD，就由另一个学 EE 的实习生照我新画的尺寸再出一版。别说，画板子门道还挺多。这 LED 有发热问题只能用铝基板，铝基板布线只能走单面，LED 功率比较大线宽又不能给太小，但因为压缩了尺寸，紧挨着 LED 的两排 M2 螺丝孔直接断绝了走线的可能。但 3D 打印件已经投出去了，只能填上一排螺丝孔走线。哎，这种活还是得既懂机械又懂电子的人来啊！

用电源夹着测了下光功率，各通道比上一版提升 50%-100% 不等。

## 荧光模块采集程序

拿电源测还不够，终究得上我们自己的采集电路试试。探头阵列装在一块 LED 板上，另一个平面再固定一块 PD 板，两块板在交叉处留焊盘再焊上。因为探头阵列整体尺寸变了，不仅 LED 板，PD 板也要重画。PD 嘉立创不给焊而且单独在一面，只能自己用焊膏拿风枪从另一面吹，所以全部件都得手焊，因为体积极小选的封装也超小，焊起来是既费手又费眼睛。因为要尽量减少漏光，3D 打印外壳给 PD 留的定位公差比较小，真正吹 PD 的时候才知道手焊这种贴片元件定位准有多难……

那个实习生那边焊着板子，我这边准备研究一下采集电路单片机的程序。因为采集电路有好几级放大，为了方便开发做了模块化设计，有单片机在的那块主控板可以通用，留了几路 SPI 从 FPC 排线接出来。但是板子打少了没有空的给我用，只好给了我一个更早的引脚不同的版本，而且同事居然把这版程序搞丢了…… 好在原理图还在，也不是不能重新写，只是之前只会 Arduino AVR 树莓派那些，STM32 还是第一次接触……

简单看了看，有什么寄存器版，标准库版，HAL 版，看着就头大。电子工程师同事跟着正点原子学下来的。该说不说教程是挺详细，不过时间紧任务重，学到 SPI 那边怎么也得两三周。STM32 这方面。正好入职之前在看 Rust，听说嵌入式也能用，查了一下 STM32F103 的支持也不错，那为啥不试一下呢。直接 [quick-start](https://github.com/rust-embedded/cortex-m-quickstart) 走起。改下编译配置，瞬间就出来一个 semihosting 打印 helloworld。再从 [f1xx-hal](https://docs.rs/stm32f1xx-hal/latest/stm32f1xx_hal/) example 里抄个点灯，前后也就十来分钟。一看 SPI 也有例程可抄，不过得先研究一下我们 ADC 和 DAC 的手册。

好家伙，前后一共没用了一天半，我这个采集测试程序直接能跑了，震惊同事一整天。Rust 在包管理这块真是吊打 c 语言生态啊。除了打印还是 semihosting 没换成串口，有阻塞和速度问题比较烦人。

## 荧光采集模块主控固件

紧接着决定整合另外一块板的功能，两个单片机变一个，电机和荧光一起控制，芯片也重新选了 F407，所以程序自然也得重新开始。单位有原子的 407 开发板，还是一样 cargo 导入 hal，点灯一气呵成。因为需要串口通信，还有实时性要求，又研究了下串口中断。一中断就打破了正常的控制流，随之带来潜在的并发问题。又因为 Rust 主打的线程安全，写中断要来回包好几层，一个变量光类型就恨不得半个窗口宽，实在有点丑陋，所以调研了几个 rust RTOS，结果没一个能用的。最后退而求其次选择了 [rtic](https://crates.io/crates/cortex-m-rtic) ~~1.0 了好耶！~~。但测试了一下串口响应发现比 uC/OS 慢不少，加上调步进电机的 CAN 控制器又遇到了些问题，还有我走了之后 Rust 代码无人维护等等缺陷，最后还是花了大半个月学 uC/OS 和 STM32 hal 库。这段时间真是进步飞快，每天全身心地投在学习里，uC/OS 英文文档来回读了好几遍，结合原子/野火各方例程边学边实践，信号量、队列、内存管理等等概念虽然不能说理解了底层原理，但基本的使用还是不成问题的，还仔细学习了一下上下文切换的核心汇编代码。

整理了一下业务逻辑，仿照内核搞了个任务分发器。串口中断处理完，先进任务分发器，根据消息决定要安排的任务和优先级（不一定回到被打断的任务）。同时还搞了个低优先级的串口发送任务用于回复上位机。我觉得逻辑还蛮清楚的，可同事说这样适合任务复杂的场景，我们这块用不到，却也没说具体怎么调整……

因为花了很多时间学习，真正业务代码开发就变得非常紧张，加上电机 CAN 驱动器居然有 BUG，停止状态位居然有可能不准，需要额外的定时任务轮询电机状态…… 连续加班半个月~~我就是个月薪 3000 的实习生啊喂~~终于赶出了个可用的版本，然后发烧病了三天。等我下周再看到同事改过的代码后，我算是理解了所谓“不用这么复杂”的意思：

- 跳过分发器任务间直接通信，优先级逻辑要在代码里到处跳才能搞明白
- 重复逻辑直接复制粘贴绝不循环：8 个信号量 8 个任务 8 个队列，串口任务里 8 个 switch case。反正业务定死了就这么多绝不写成数组

行吧，反正他改过了，那就算他接手了，领导给我派别的任务了

## C++/Qt 数据拟合 + 可视化

别说，PCR S 形扩增曲线这个事儿我高中还是学过的。反倒是同事们都是 90 年左右出生，高中的时候 PCR 还没进课本，入职培训听的一愣一愣的。软工同事 GitHub 找了个 GSL 库曲线拟合的 C++ wrapper ~~GPL 代码真的合适么~~，虽然初始猜测给的很离谱，但双曲正切还是拟出来了而且效果不错。

核酸检测结果需要计算 [CT]^(Cycle Threshold) 值，我们希望软件能够自动计算。流程大概是先拟合，然后定基线范围，按基线平均值 + k 倍基线标准差画一根水平线，取该线和曲线交点 x 值作为 CT 值。基本原理并不难，但实际应用时有各种问题。

- 拟合异常：

  至少开发阶段，不扩增的情况很常见，甚至有时候曲线还能往下掉，这时候拟合就会很离谱。用拟合前一个预判断 + 拟合后参数校验即可

- 各通道基线对齐：

  这个其实好说，拟出来参数之后重新算下 y 偏移就行

- 通道串扰：

  这个比较复杂，需求临近实习结束才提出来，所以我只给出了线代的公式和实现的想法。理论上各通道都是线性的，可以提前用已知强度的荧光溶液标定出各通道的补偿矩阵，实际数据扣除基线后用逆矩阵转换一下即可。如果拟合成功，只需要对各通道的拟合参数应用该矩阵即可。

C++ Qt 虽然也是第一次接触，不过也就掉了一次内存泄漏和一次 `std::array` 的坑。此外画图的部分沿用了之前同事找的开源控件，还做了一些小小的魔改来适配我们的项目。C++ 业务逻辑写多了，才能体会到 Rust Enum 的设计精妙之处。比方说拟合失败，Rust 里面返回 `Result<T,E>` 或者 `Option<T>` 就很符合语义。但在 C++ 里想做到同样的事情就得要么手动实现 tagged union，要么传指向堆内存的可空指针给调用者检查，要么就得用异常。Tagged Union 得给同事们讲明白，传指针项目里从来没这么用过，搞不好又是内存泄漏，试了一下异常结果 Linux 工控机上怎么调编译选项也不支持。还好结果都是 float，只好传进引用然后用 NAN 表示失败了……

## 总结

虽然说是托关系找的实习，但也确实投入了相当多的实际开发，获得了很多机电软方面的实践经验。第一份实习竟然就得到了如此全面的锻炼！工作环境也相当不错，标配人体工学椅，小零食饮料以及咖啡机，采购报销虽然要自己垫钱但回款也很及时，同事领导也都很友善。

