最近，开发了一种名为VoxelNet [14]的新方法。该方法将原始点云特征提取和基于体素的特征提取结合在单级端到端网络中。它首先将点云数据分组为体素，然后通过体素应用线性网络体素，然后将体素转换为密集的3D张量，以用于区域提议网络（RPN）[16]。目前，这是一种最先进的方法。然而，其计算成本使其难以用于实时应用。在本文中，我们提出了一种称为SECOND（稀疏嵌入式卷积检测）的新方法，该方法通过最大限度地利用点云数据中存在的丰富3D信息来解决基于3D卷积的检测中的这些挑战。该方法包含对现有卷积网络架构的若干改进。引入空间稀疏卷积网络用于基于LiDAR的检测，并且用于在将3D数据下采样到类似于2D图像数据之前从z轴提取信息。我们还使用基于GPU（图形处理单元）的规则生成算法进行稀疏卷积以提高速度。与密集卷积网络相比，我们的基于稀疏卷积的检测器在KITTI数据集的训练期间实现了4倍速度增强，并且推理速度提高了3倍。作为进一步的测试，我们设计了一个用于实时检测的小型模型，在GTX 1080 Ti GPU上运行时间约为0.025秒，性能略有下降。
使用点云数据的另一个优点是，通过将直接变换应用于这些对象上的指定点，可以非常轻松地缩放，旋转和移动对象。 SECOND基于此功能采用了一种新颖的数据增强形式。生成地面实况数据库，其包含对象的属性和关联的点云数据。然后，在训练期间，从该数据库采样的对象被引入点云。这种方法可以大大提高网络的收敛速度和最终性能。
除此之外，我们还引入了一种新颖的角度损失回归方法来解决当地面实况和预测之间的方向差等于π时产生的大损失问题，这产生了与真实相同的边界框。 这种角度回归方法的性能超过了我们所知的任何现有方法，包括AVOD中可用的方向向量回归函数[9]。 我们还引入了一个辅助方向分类器来识别物体的方向。
在提交时，我们的方法为基于KITTI的3D检测[7]在所有类别中产生最先进的结果，而对于较大的模型以20 fps运行，对于较小的模型以40 fps运行。
我们工作的主要贡献如下：
- 我们在基于LiDAR的物体检测中应用稀疏卷积，从而大大提高了训练和推理的速度。
- 我们提出了一种改进的稀疏卷积方法，使其运行得更快。
- 我们提出了一种新的角度损失回归方法，它表明了更好的方向回性能比其他方法做的。
- 我们介绍了一种新的数据增强方法，用于仅限LiDAR的学习问题，大大提高了收敛速度和性能。
2.1。 基于前视图和图像的方法
使用RGB-D数据的2D表示的方法可以分为两类：基于鸟瞰图（BEV）的那些和基于正视图的那些。 在典型的基于图像的方法[5]中，首先生成2D边界框，类语义和实例语义，然后使用手工制作的方法生成特征映射。 另一种方法[17]使用CNN从图像估计3D边界框和特殊设计的离散连续CNN来估计对象的方向。 使用LiDAR的方法[18]涉及将点云转换为前视2D图和应用2D检测器来定位前视图像中的对象。 与其他方法相比，这些方法对BEV检测和3D检测都表现不佳。
2.2。 基于鸟瞰图的方法
MV3D [8]是第一种将点云数据转换为BEV表示的方法。 在该方法中，将点云数据转换为若干切片以获得高度图，然后将这些高度图与强度图和密度图连接以获得多通道特征。 ComplexYOLO [19]使用YOLO（You Only Look Once）[20]网络和复杂角度编码方法来提高速度和方向性能，但它在预测的3D边界框中使用固定高度和z位置。 在[21]中，设计了一种快速的单级无提议探测器，它利用特定的高度编码BEV输入。 然而，所有这些方法的关键问题是在生成BEV图时许多数据点被丢弃，导致垂直轴上的信息大量丢失。 此信息丢失严重影响这些方法在3D边界框回归中的性能。
2.3。基于3D的方法
大多数基于3D的方法要么直接使用点云数据，要么需要将这些数据转换为3D网格或体素，而不是生成BEV表示。在[12]中，将点云数据转换为包含特征向量的体素，然后使用新的基于卷积的基于投票的算法进行检测。参考。 [13]利用以特征为中心的投票方​​案来实现新的卷积，从而提高计算速度，从而利用点云数据的稀疏性。这些方法使用手工制作的功能，虽然它们在特定数据集上产生令人满意的结果，但它们无法适应自动驾驶中常见的复杂环境。在一个独特的方法中，[22,23]的作者开发了一种系统，可以通过基于CNN的新型架构直接从点云学习逐点特征，而参考文献。 [24]使用k邻域方法和卷积来学习点云的局部空间信息。这些方法直接处理点云数据以在k邻域点上执行一维卷积，但它们不能应用于大量点;因此，需要图像检测结果来过滤原始数据点。一些基于CNN的检测器将点云数据转换为体素。在[15]中提出的方法中，将点云数据离散化为二值体素，然后应用3D卷积。 [14]的方法将云数据分组为体素，提取体素特征，然后将这些特征转换为密集张量，以使用3D和2D卷积网络进行处理。这些方法的主要问题是3D CNN的高计算成本。不幸的是，3D CNN的计算复杂度随着体素分辨率而立方增长。在[25,26]中，设计了一个空间稀疏卷积，可以提高3D卷积速度，而Ref。 [27]提出了一种新的3D卷积方法，其中输出的空间结构保持不变，这大大提高了处理速度。在[28]中，子流形卷积被应用于3D语义分割任务;但是，没有已知的方法使用稀疏卷积进行检测任务。与所有这些方法类似，我们的方法使用3D卷积架构，但它包含了几个新的改进。
2.4。基于融合的方法
一些方法将相机图像与点云相结合。例如，[29]的作者使用具有不同感受野的两个尺度的3D RPN来生成3D提议，然后将3D体积从每个3D提议的深度数据馈送到3D CNN并将相应的2D颜色补丁投入到2D CNN预测最终结果。在[8]中提出的方法中，将点云数据转换为前视图和BEV，然后从两个点云图提取的特征图与图像特征图融合。具有图像的MV3D网络比仅使用BEV的网络表现更好，但是这种体系结构不适用于小型对象并且运行缓慢，因为它包含三个CNN。 [9]的作者将图像与BEV相结合，然后使用新颖的架构生成高分辨率特征图，3D物体2D检测结果用于过滤点云，以便PointNet [22]可用于预测3D边界框。然而，基于融合的方法通常运行缓慢，因为它们需要处理大量的图像输入。具有LiDAR功能的时间同步和校准相机的额外要求限制了可以使用这些方法的环境并降低了它们的稳健性。相比之下，我们的方法可以仅使用LiDAR数据实现最先进的性能。
3.second检测器
在本节中，我们描述了所提出的SECOND探测器的架构，并提供了有关训练和推理的相关细节。
3.1。 网络架构
拟议的SECOND探测器如图1所示，由三个部分组成：
（1）体素特征提取器; （2）稀疏卷积中间层; （3）RPN。
3.1.1。点云分组
在这里，我们遵循[14]中描述的简单过程来获得点云​​数据的体素表示。我们首先根据指定的体素数量限制来预先分配缓冲区;然后，我们迭代点云并将点分配给它们相关的体素，我们保存体素坐标和每个体素的点数。我们在迭代过程中根据哈希表检查体素的存在。如果与点相关的体素尚不存在，我们在哈希表中设置相应的值;否则，我们将体素的数量增加1。一旦体素的数量达到指定的限制，迭代过程将停止。最后，我们获得所有体素，它们的坐标和每个体素的点数，用于实际的体素数量。为了检测相关类别中的汽车和其他物体，我们根据z - y处的[ - 3,1]×[ - 40,40]×[0,70.4] m的地面实况分布来绘制点云。 ×x轴。对于行人和骑车人检测，我们使用[ - 3,1]×[ - 20,20]×[0,48] m处的裁剪点。对于我们较小的模型，我们仅使用[ - 3,1]×[ - 32,32]×[0,52.8] m范围内的点来增加推理速度。需要根据体素大小稍微调整裁剪区域，以确保生成的特征图的大小可以在后续网络中正确地下采样。对于所有任务，我们使用v D = 0.4×v H = 0.2×v W = 0.2 m的体素大小。用于汽车检测的每个空体素中的最大点数被设置为T = 35，其基于KITTI数据集中每个体素的点数的分布来选择;行人和骑车者检测的相应最大值设置为T = 45，因为行人和骑车者相对较小，因此，体素特征提取需要更多的点。
3.1.2。 体素特征提取器
我们使用体素特征编码（VFE）层，如[14]中所述，提取体素特征.VFE层将同一体素中的所有点作为输入，并使用由线性层组成的完全连接网络（FCN）， 批量标准化（BatchNorm）层和整流线性单元（ReLU）层，以提取逐点特征。 然后，它使用元素最大池来获得每个体素的本地聚合特征。 最后，它对获得的特征进行平铺，并将这些平铺特征和逐点特征连接在一起。 我们使用VFE（c out）来表示将输入特征转换为c个外向输出特征的VFE层。 类似地，FCN（c out）表示Linear-BatchNorm-ReLU层，它将输入要素转换为c个外向输出要素。 整体而言，体素特征提取器由多个VFE层和FCN层组成。