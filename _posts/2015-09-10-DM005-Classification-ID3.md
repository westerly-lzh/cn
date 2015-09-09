---
layout: post
title:  数据挖掘系列005 数据分类 决策树 ID3
categories:
- data mining
tags:
- 数据分类
- 决策树
- ID3

---

数据分类是通过分析输入数据，利用输入数据的部分属性构造一个分类函数或者分类模型，利用该函数或者模型把数据库中的其他数据记录映射到给定的类别中。分类可以用于预测。
在解决分类问题中，决策树方法是一种行之有效的方法。目前已有的决策树算法包括但不限于：ID3,CHAID,CART,FACT,C4.5,SLIQ和SPRINT等。

##导言

决策树算法是通过分析输入的数据属性，构造一颗有向无环树，该树的每个叶子节点都对应数据记录的类别标识的一个值，每个内部节点都对应了数据记录的一个属性，父子节点之间的连线都对应了属性的一个分割。决策树算法一般可分为两个阶段：1）决策树生成，2）决策树修剪。本系列主要讨论决策树的生成。

## 信息论基础

决策树生成算法的关键是求解属性的分割方法。目前的属性分割方法主要包括两类：基于信息论的方法和基于最小Gini指标的方法。前者对应ID3，C4.5；后者对应CART，SLIQ和SPRINT。本文主要讨论ID3算法及其对应的基于信息论的分割方法。

**定义1**：若给定的概率分布 $P=(p_1,P_2,...,p_n)$ ,则由该分布传递的信息量称为P的熵，即：

$$I(P) = -(p_1 \times log_2(p_1)+ p_2 \times log_2(p_2) + ... + p_n \times log_2(p_n))$$

**定义2**:若一个给定的记录集合T根据类别属性，被分成相互独立的类 $C_1,C_2,...,C_k$ ,则识别T的一个记录所属的类别所需要的信息量是$info(T) = I(P)$ ,其中P表示
$(C_1,C_2,...,C_k)$ 的概率分布，即：
  
 $$P=(\frac{\|C_1\|}{\|T\|},\frac{\|C_2\|}{\|T\|},...,\frac{\|C_k\|}{\|T\|})$$

**定义3**:若先根据非类别属性X的值将T分成集合$T_1,T_2,...,T_n$，则确定T中一个记录所属类别的信息量可以通过确定 $T_i$ 的加权平均得到，即：

$$info(X,T) = \sum_{i=1}^n (\frac{\|T_i\|}{\|T\|}) \times info(T_i)$$


**定义4**： 定义信息增益$Gain(X,T)$为:

$$Gain(X,T) = info(T) - info(X,T)$$

信息增益表示了两个信息量之间的差值，其中一个信息量的确定需要T，另一个信息量是在已知属性X的值后确定T的一条记录的信息量，所以信息增益与属性X相关。每次分割属性时，选择信息增益最大的属性。

## ID3算法

ID3算法构造决策树的过程如下：

![image](/media/img/algori/id3.png)

## ID3简单实现

下面的代码是ID3生成决策树算法采用Java语言的简单实现：

	package classify;

	import java.util.ArrayList;
	import java.util.HashMap;
	import java.util.List;
	import java.util.Map;

	/**
	 * 简单的ID3算法实现
	 * 该实现适用于属性值为离散变量的决策树生成。
	 * @author jeff
	 *
	 */
	public class ID3 {

		/**
		 * 决策树的数据结构
		 * @author jeff
		 *
		 */
		public class TreeNode {
			private int index;//该节点在数据集合中所代表的属性的下标
			private String name;//该节点在数据结集合中所代表的属性名称
			private TreeNode parent;//父节点
		
			private Integer leftValue;//当节点为子节点时，用来存储节点的值
		
			private String linkLabel;//在子节点存储link信息
			private List<TreeNode> children;//该节点的子节点

			public int getIndex() {
				return index;
			}

			public void setIndex(int index) {
				this.index = index;
			}

			public String getName() {
				return name;
			}

			public void setName(String name) {
				this.name = name;
			}

			public TreeNode getParent() {
				return parent;
			}

			public void setParent(TreeNode parent) {
				this.parent = parent;
			}

			public List<TreeNode> getChildren() {
				return children;
			}

			public void setChildren(List<TreeNode> children) {
				this.children = children;
			}

			public void addChild(TreeNode child) {
				if (this.children == null) {
					children = new ArrayList<ID3.TreeNode>();
				}
				children.add(child);
			}

			public String getLinkLabel() {
				return linkLabel;
			}

			public void setLinkLabel(String linkLabel) {
				this.linkLabel = linkLabel;
			}

			public Integer getLeftValue() {
				return leftValue;
			}

			public void setLeftValue(Integer leftValue) {
				this.leftValue = leftValue;
			}

		}

		public TreeNode buildTree(RecordSet set) {
			TreeNode root = new TreeNode();
			classify(root, set);

			return root;
		}

		/**
		 * 递归生成决策树
		 * 
		 * @param root
		 * @param current
		 */
		public void classify(TreeNode root, RecordSet current) {
			if (!current.checkValid()) {
				TreeNode node = new TreeNode();
				node.setName("Failure");
				node.setIndex(-1);
				node.setParent(root);
				root.addChild(node);
				return;
			}
			if (current.check()) {
				TreeNode node = new TreeNode();
				node.setName(current.getName(current.getCategoryIndex()));
				node.setIndex(current.getCategoryIndex());
				node.setParent(root);
				node.setLinkLabel(root.getName()+"_"+current.getKeyValue()+"_"+node.getName());
				node.leftValue = current.getValue(0, node.getIndex());
				root.addChild(node);
				return;
			}

			double maxGain = Double.MIN_VALUE;
			Integer maxIndex = -1;
			List<RecordSet> sets = new ArrayList<RecordSet>();

			Integer[] indexs = current.getIndexs();
			for (int i = 0; i < indexs.length; i++) {
				if (indexs[i].intValue() == current.getCategoryIndex()) {
					continue;
				}
				List<RecordSet> tmpSets;
				Map<Integer, List<Integer[]>> maxIndexSplit = new HashMap<Integer, List<Integer[]>>();
				for (Integer[] values : current.getValues()) {
					List<Integer[]> set = maxIndexSplit.putIfAbsent(values[indexs[i]], new ArrayList<Integer[]>());
					if (set != null) {
						set.add(values);
					} else {
						maxIndexSplit.get(values[indexs[i]]).add(values);
					}
				}

				tmpSets = makeRecordSet(current, maxIndexSplit);

				double gain = computeGain(current, tmpSets);
				sets = maxGain < gain ? tmpSets : sets;
				maxIndex = maxGain < gain ? i : maxIndex;
				maxGain = maxGain < gain ? gain : maxGain;
			}

			TreeNode node = new TreeNode();
			node.index = indexs[maxIndex];
			node.name = current.getName(indexs[maxIndex]);
			node.parent = root;
			node.setLinkLabel(root.getName()+"_"+current.getKeyValue()+"_"+node.getName());
			root.addChild(node);

			assert sets != null;
			for (RecordSet set : sets) {
				set.getIndexs()[maxIndex] = -1;
				set = set.filter();
				classify(node, set);
			}

		}

		/**
		 * 根据属性值的不同，对数据集合进行划分，并对每个划分生成新的数据子集
		 * @param current
		 * @param res
		 * @return
		 */
		private List<RecordSet> makeRecordSet(RecordSet current, Map<Integer, List<Integer[]>> res) {
			List<RecordSet> sets = new ArrayList<RecordSet>();
			for (Map.Entry<Integer, List<Integer[]>> entry:res.entrySet()) {
				RecordSet set = new RecordSet();
				set.setIndexs(current.getIndexs());
				set.setIndexName(current.getIndexName());
				set.setValues((Integer[][]) entry.getValue().toArray(new Integer[][] {}));
				set.setCategoryIndex(current.getCategoryIndex());
				set.setRowNum(entry.getValue().size());
				set.setKeyValue(entry.getKey());
				sets.add(set);
			}
			return sets;
		}

		/**
		 * 计算信息熵
		 * @param set
		 * @return
		 */
		private double computeInfo(RecordSet set) {
			Map<Integer, Integer> counter = new HashMap<Integer, Integer>();
			for (Integer[] values : set.getValues()) {
				Integer value = counter.putIfAbsent(values[set.getCategoryIndex()], 1);
				if (value != null) {
					counter.replace(values[set.getCategoryIndex()], value + 1);
				}
			}
			List<Double> pros = new ArrayList<Double>();
			for (Integer value : counter.values()) {
				pros.add(value * 1.0 / set.getRowNum());
			}

			return Util.entropy((Double[]) pros.toArray(new Double[] {}));
		}

		/**
		 * 计算信息增益
		 * @param current
		 * @param subSets
		 * @return
		 */
		private double computeGain(RecordSet current, List<RecordSet> subSets) {

			double infoTbyX = 0.0;
			for (RecordSet recordSet : subSets) {
				infoTbyX = infoTbyX + (recordSet.getRowNum() / current.getRowNum()) * computeInfo(recordSet);
			}
			double infoT = computeInfo(current);
			return infoT - infoTbyX;
		}

		public static void main(String[] args) {
			RecordSet set = new RecordSet();
			set.setCategoryIndex(2);
			set.setIndexName(new String[] { "Age", "Type", "Risk" });
			set.setIndexs(new Integer[] { 0, 1, 2 });
			set.setRowNum(6);
			set.setValues(
					new Integer[][] { { 23, 1, 1 }, { 17, 2, 1 }, { 43, 2, 1 }, { 68, 1, 2 }, { 32, 3, 2 }, { 20, 1, 1 } });
			set.setKeyValue(6);
		
		
		
			RecordSet set1 = new RecordSet();//股票训练数据
			set1.setCategoryIndex(3);
			set1.setIndexName(new String[]{"年龄","竞争力","类型","收益"});
			set1.setIndexs(new Integer[]{0,1,2,3});
			set1.setRowNum(10);
			/**
			 * 年龄			竞争力		类型		收益
			 * 1：老年		1：有		1：软件	1：下降
			 * 2：中年		2：无		2：硬件	2：上升
			 * 3：青年	
			 * 
			 * 
			 * 
			 * 
			 */
		
			set1.setValues(new Integer[][]{
				{1,1,1,1},
				{1,2,1,1},
				{1,2,2,1},
				{2,1,2,1},
				{2,1,2,1},
				{2,2,2,2},
				{2,2,1,2},
				{3,1,1,2},
				{3,2,2,2},
				{3,2,2,2}});
		
			ID3 id3 = new ID3();
			TreeNode tree = id3.buildTree(set1);
			System.out.println(tree);
		}

	}
	

其中使用到的RecordSet类定义如下：

	package classify;

	import java.util.ArrayList;
	import java.util.List;

	/**
	 * 离散类别 ID3 算法 数据模型
	 * 
	 * @author jeff
	 *
	 */
	public class RecordSet {

		private Integer keyValue; //RecordSet按属性值进行划分子集时，属性的值

		private Integer[] indexs; // 还未被划分的属性下标
		private String[] indexName; //属性的名称
		private Integer[][] values; //集合值
		private Integer categoryIndex;//分类下标
		private Integer rowNum; //该集合中总的行数

		public boolean removeIndex(Integer index) {
			if (index >= 0 && index < indexs.length) {
				indexs[index] = -1;
				return true;
			}
			return false;
		}

		public String getName(Integer index) {
			for (int i = 0; i < indexs.length; i++) {
				if (index.intValue() == indexs[i]) {
					return indexName[i];
				}
			}
			return null;
		}

		public Integer getValue(Integer rowIndex, Integer colIndex) {
			return values[rowIndex][colIndex];
		}

		public Integer[] getIndexs() {
			return indexs;
		}

		public void setIndexs(Integer[] indexs) {
			this.indexs = indexs;
		}

		public String[] getIndexName() {
			return indexName;
		}

		public void setIndexName(String[] indexName) {
			this.indexName = indexName;
		}

		public Integer[][] getValues() {
			return values;
		}

		public void setValues(Integer[][] values) {
			this.values = values;
		}

		public boolean checkValid() {
			return this.indexs != null && this.indexName != null && this.indexs.length == this.indexName.length
					&& indexs.length > 1 && values.length > 0;
		}

		public boolean check() {
			Integer firstValue = values[0][this.categoryIndex];
			for (Integer[] value : this.values) {
				if (value[this.getCategoryIndex()] != firstValue) {
					return false;
				}
			}
			return true;

		}

		public Integer getCategoryIndex() {
			return categoryIndex;
		}

		public void setCategoryIndex(Integer categoryIndex) {
			this.categoryIndex = categoryIndex;
		}

		public Integer getRowNum() {
			return rowNum;
		}

		public void setRowNum(Integer rowNum) {
			this.rowNum = rowNum;
		}

		public RecordSet filter() {
			List<Integer> new_index = new ArrayList<Integer>();
			List<String> new_names = new ArrayList<String>();
			Integer new_categoryIndex = this.categoryIndex;
			RecordSet set = new RecordSet();

			for (int i = 0; i < indexs.length; i++) {
				if (indexs[i] >= 0) {
					new_index.add(indexs[i]);
					new_names.add(indexName[i]);
				}
				if (indexs[i].intValue() == this.categoryIndex) {
					new_categoryIndex = indexs[i];
				}
			}

			set.keyValue = this.keyValue;
			set.indexs = (Integer[]) new_index.toArray(new Integer[] {});
			set.indexName = (String[]) new_names.toArray(new String[] {});
			set.values = this.values;
			set.categoryIndex = new_categoryIndex;
			set.rowNum = this.rowNum;
			return set;
		}

		public Integer getKeyValue() {
			return keyValue;
		}

		public void setKeyValue(Integer keyValue) {
			this.keyValue = keyValue;
		}

	}
	
	
熵的计算如下：

	package classify;

	public class Util {

		public static double entropy(Double[] pros) {
			double sum = 0.0;
			double log10_2 = Math.log10(2);
			for (double d : pros) {
				sum = d * Math.log10(d) / log10_2;
			}
			return -sum;
		}

	}
	

## 讨论


* 本实现的ID3算法只适用于离散属性。对于连续属性的决策树构建，需要先对数据集进行离散化处理。
* 本实现在递归过程中，需要每次递归都需要遍历整个数据集，效率不高。
