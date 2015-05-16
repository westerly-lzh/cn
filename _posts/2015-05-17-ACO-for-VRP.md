---
layout: post
title:  循环取货问题的蚁群算法
categories:
- 优化
tags:
- 蚁群算法
- VRP问题


---

## 引子

一个朋友在循环取货（Milk-run）问题中需要使用蚁群算法，于是我就特地的找了一篇讨论该问题的论文《一种动态循环取货的入场物流策略及其线路规划研究》，并实现了该文献中讨论的算法。

## 数据准备

	Vvv	v00	v01	v02	v03	v04	v05	v06	v07	v08	v09	v010	q	t
	v00	0	28	41	20	41	30	22	50	22	28	31	0	0
	v01	28	0	22	44	60	22	41	60	30	40	58	15	0.6
	v02	41	22	0	50	58	14	60	82	50	36	64	9	0.4
	v03	20	44	50	0	22	36	36	64	41	20	14	12	0.5
	v04	41	60	58	22	0	44	58	86	63	22	22	14	0.6
	v05	30	22	14	36	44	0	50	76	44	22	50	16	0.6
	v06	22	41	60	36	58	50	0	28	14	50	41	8	0.3
	v07	50	60	82	64	86	76	28	0	31	78	67	13	0.5
	v08	22	30	50	41	63	44	14	31	0	50	50	15	0.6
	v09	28	40	36	20	22	22	50	78	50	0	31	7	0.3
	v10	31	58	64	14	22	50	41	67	50	31	0	8	0.3

该问题中有十个供应商，分别为v01 ~ v10, v00为配送中心，q为车辆在每个供应商处的载货量（按照体积计算），而t为在该供应商处的集货时间。车辆限制为：车辆最大载货体积为41 $$$m^3$$$, 车辆的行驶距离为200km，车速为40，车辆最大行驶时间限制为7小时。

## 模型

假定供应商以$$$V_i$$${$$$i=0,1,2,...,n$$$}来表示，其中$$$V_0$$$代表集货中心，集货中心有$$$m$$$辆车，所有车的行驶速度一致，车辆的最大载货量表示为$$$Q\_{max}$$$，车辆允许的最大行驶时间为$$$T\_{max}$$$,允许的最大行驶距离为$$$L\_{max}$$$,车辆的行驶速度为$$$V$$$，供应商$$$V\_i$$$与供应商$$$V\_j$$$之间的距离用$$$d\_{ij}$$$,供应商$$$V\_i$$$到供应商$$$V\_j$$$需要花费的时间为$$$\frac{d\_{ij}}{V}$$$,在供应商$$$V\_i$$$处的集货时间为$$$t\_{ci}$$$,发送量为$$$q\_i$$$.

定义：$$$X\_{ijk} =\cases{{1}&车辆k从供应商V_i行驶到供应商V_j \\\\{0}&否则} $$$,$$$y\_{ik} = \cases{{1}&供应商V\_i由车辆k进行循环取货 \\\\{0}&否则}$$$。则该模型可以表示如下：

$$D = min(\sum\_{i=0}^n\sum\_{j=0}^n\sum\_{k=1}^md\_{ij}X\_{ijk}) \label{1}$$

$$\sum\_{k=1}^m y\_{ik} =1  \quad  i=1,2,...,n \label{2}$$

$$\sum\_{i=0}^n\sum\_{k=1}^m X\_{ijk} =1  \quad  j=1,2,...,n \label{3}$$

$$\sum\_{j=1}^n\sum\_{k=1}^m X\_{ijk} =1  \quad  i=1,2,...,n \label{4}$$

$$\sum\_{i=1}^n\sum\_{k=1}^m X\_{oik} =\sum\_{j=1}^n\sum\_{k=1}^mX\_{jok} \label{5}$$

$$\sum\_{i=1}^ny\_{ik}q\_i \le Q\_{max} \quad k=1,2,...,m \label{6}$$

$$\sum\_{i=1}^n\sum\_{j=1}^nX\_{ijk}d\_{ij} \le L\_{max} \quad k=1,2,...,m \label{7}$$

$$\sum\_{i=0}^n\sum\_{j=1}^n X\_{ijk}(\frac{d\_{ij}}{V}+t\_{cj}) \le T\_{max} \quad k=1,2,...,m \label{8}$$


## 蚁群算法原理
蚂蚁在觅食过程中，在第一次经过一个还没有走过的路径的分叉口时，会随机的挑选其中一条路径前进，并分泌出与路径信息相关的信息素，当后来的蚂蚁再次经过该岔路口时就会通过感知已经经过该路径的蚂蚁遗留下来的信息素，并通过选择信息素浓度较大的路径来完成寻觅工作。这种正向的反馈机制能够有效的引导蚂蚁找到最优路径，因为最优路径上经过的蚂蚁会越来越多，信息量就会越来越大，而由于非最优路径上的信息量会随着时间的流逝而逐渐减少直至消失。


## Java 实现

Ant.java

	import java.util.Random;
	import java.util.Vector;

	public class Ant implements Cloneable {

		private Vector<Integer> tabu;// 禁忌表
		private Vector<Integer> allowedCities;// 允许搜索的城市
		private float[][] delta;// 信息素变化矩阵
		private float[][] distance;// 距离矩阵
		
		private float alpha;//
		private float beta;//
		private float lamda;
		private float omga;
		
		private int tourLength;// 路径长度
		private int cityNum;// 城市数量
		
		private int firstCity;// 起始城市
		private int currentCity;// 当前城市
		
		// /Added
		
		// private int[] tour;
		private float[] Q;// 供应商的发送量
		private float[] T;// 供应商的集货时间
		private float V; // 车辆速度
		
		private float Q_max;
		private float L_max;
		private float T_max;
		
		private float load_Q = 0;
		private float load_T = 0;
		private float load_D = 0;
		
		public void resetLoad() {
			load_Q = 0.0f;
			load_T = 0.0f;
			load_D = 0.0f;
		}

		public boolean selectCityIsOK(int selectCity) {
			return ((load_Q + Q[selectCity]) <= Q_max) && ((load_D + distance[tabu.get(tabu.size() - 2)][selectCity]) <= L_max)
					&& ((T[selectCity] + distance[tabu.get(tabu.size() - 2)][selectCity] / V) <= T_max);
		}
		
		public void updateLoadQ(int selectCity) {
			load_Q += load_Q + Q[selectCity];
			load_D += load_D + distance[tabu.get(tabu.size() - 2)][selectCity];
			load_T += load_T + T[selectCity] + distance[tabu.get(tabu.size() - 2)][selectCity] / V;
		}
		
		public Ant() {
			cityNum = 30;
			tourLength = 0;
			Q = new float[cityNum];
			T = new float[cityNum];
			V = 40;
		
			Q_max = 41;
			L_max = 200;
			T_max = 7.0f;
		
		}
		
		public Ant(int cityNum, float[] Q, float[] T, float V, float Q_max, float L_max, float T_max) {
			this.cityNum = cityNum;
			tourLength = 0;
			this.Q = Q;
			this.T = T;
			this.V = V;
		
			this.Q_max = Q_max;
			this.L_max = L_max;
			this.T_max = T_max;
		}
		
		/**
	 	* 初始化蚂蚁，所有蚂蚁必须从配送中心出发，配送中心为0
	 	* 
		* @param distance
	 	* @param alpha
	 	* @param beta
	 	*/
		public void init(float[][] distance, float alpha, float beta, float lamda, float omga) {
			this.alpha = alpha;
			this.beta = beta;
			this.lamda = lamda;
			this.omga = omga;
			allowedCities = new Vector<Integer>();
			tabu = new Vector<Integer>();
			this.distance = distance;
			this.delta = new float[cityNum][cityNum];
			for (int i = 0; i < cityNum; i++) {
				allowedCities.add(new Integer(i));
				for (int j = 0; j < cityNum; j++) {
					this.delta[i][j] = 0.0f;
				}
			}
			// -------------BEGIN
			// 蚂蚁随机选择初始化的城市 FIXME，新应用中应当使蚂蚁从配送中心出发
			// Random random = new Random(System.currentTimeMillis());
			// firstCity = random.nextInt(cityNum);
			firstCity = 0;
			// --------------END
			for (Integer i : allowedCities) {
				if (i.intValue() == firstCity) {
					allowedCities.remove(i);
					break;
				}
			}
			tabu.add(Integer.valueOf(firstCity));
			currentCity = firstCity;
			resetLoad();
		}
	
		/**
		* 选择下一个城市
	 	* 
	 	* @param pheromone
	 	*            信息素
	 	*/
		public int selectNextCity(float[][] pheromone) {
			double[] p = new double[cityNum];
			double sum = 0.0f;
			// 计算分母部分
			for (Integer i : allowedCities) {
				sum += Math.pow(pheromone[currentCity][i.intValue()], alpha) * Math.pow(1.0 / distance[currentCity][i.intValue()], beta)
						* Math.pow((load_Q + Q[i.intValue()]) / Q_max, lamda)
						* Math.pow((L_max / (load_D + distance[currentCity][i.intValue()] + distance[i.intValue()][0])), omga);
			}
			// 计算概率矩阵
			for (int i = 0; i < cityNum; i++) {
				boolean flag = false;
				for (Integer j : allowedCities) {
					if (i == j.intValue()) {
						p[i] = (Math.pow(pheromone[currentCity][i], alpha) * Math.pow(1.0 / distance[currentCity][i], beta)
								* Math.pow((load_Q + Q[j.intValue()]) / Q_max, lamda) * Math.pow(
								(L_max / (load_D + distance[currentCity][j.intValue()] + distance[j.intValue()][0])), omga))
								/ sum;
						flag = true;
						break;
					}
				}
				if (!flag) {
					p[i] = 0.0f;
				}
			}
			// 轮盘赌选择下一个城市
			Random random = new Random(System.currentTimeMillis());
			float selectP = random.nextFloat();
			int selectCity = 0;
			float sum1 = 0.0f;
			for (int i = 0; i < cityNum; i++) {
				sum1 += p[i];
				if (sum1 > selectP) {
					selectCity = i;
					break;
				}
			}
			// 从允许的城市中去掉selectCity
			for (Integer i : allowedCities) {
				if (i.intValue() == selectCity) {
					allowedCities.remove(i);
					break;
				}
			}
		
			// 在禁忌表中添加selectCity
			tabu.add(Integer.valueOf(selectCity));
			// 将当前城市改为选择城市
			currentCity = selectCity;
			return selectCity;
		}
		
		public int calculateTourLength() {
			int len = 0;
			for (int i = 0; i < tabu.size() - 1; i++) {
				len += distance[this.tabu.get(i).intValue()][this.tabu.get(i + 1).intValue()];
			}
			return len;
		}
	}


ACO.java

	import java.io.BufferedReader;
	import java.io.FileInputStream;
	import java.io.IOException;
	import java.io.InputStreamReader;
	import java.util.Arrays;
	import java.util.Vector;
	import java.util.function.Consumer;
	
	public class ACO {
	
		private Ant[] ants;
		private int antNum;
		private int cityNum;
		private int MAX_GEN;
		private float[][] pheromone;
		private float[][] distance;
		private float bestLength;
	
		private float alpha;
		private float beta;
		private float lamda;
		private float omga;
		private float rho;
	
		private float[] Q;// 供应商的发送量
		private float[] T;// 供应商的集货时间
		private float V = 40; // 车辆速度
	
		private float Q_max = 41;
		private float L_max = 200;
		private float T_max = 7.0f;
	
		public ACO() {
	
		}

		public ACO(int antNum, int max_Gen, float alpha, float beta, float lamda, float omga, float rho) {
			this.antNum = antNum;
			this.MAX_GEN = max_Gen;
			this.ants = new Ant[antNum];
			this.alpha = alpha;
			this.beta = beta;
			this.lamda = lamda;
			this.omga = omga;
			this.rho = rho;
		}
	
		private float[] parseFloat(String[] sigs) {
			float[] res = new float[sigs.length];
			for (int i = 0; i < sigs.length; i++) {
				res[i] = Float.valueOf(sigs[i]);
			}
			return res;
		}
	
		public void init(String fileName) throws IOException {
			BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream(fileName)));
			String buff;
			buff = reader.readLine();
			String[] segments = buff.split("	");
			cityNum = segments.length - 3;
			distance = new float[cityNum][cityNum];
			Q = new float[cityNum];
			T = new float[cityNum];
			
			for (int i = 0; i < cityNum; i++) {
				buff = reader.readLine();
				segments = buff.split("	");
				distance[i] = parseFloat(Arrays.copyOfRange(segments, 1, 12));
				Q[i] = Integer.valueOf(segments[segments.length - 2]);
				T[i] = Float.valueOf(segments[segments.length - 1]);
			}
			// 初始化信息素
			pheromone = new float[cityNum][cityNum];
			for (int i = 0; i < cityNum; i++) {
				for (int j = 0; j < cityNum; j++) {
					pheromone[i][j] = 0.1f;
				}
			}
		
			bestLength = Integer.MAX_VALUE;
		
			// 随机放置蚂蚁
			for (int i = 0; i < antNum; i++) {
				ants[i] = new Ant(cityNum, Q, T, V, Q_max, L_max, T_max);
				ants[i].init(distance, alpha, beta, lamda, omga);
			}
			reader.close();
		}
		
		public class Wrap{
			public int gen;
		}
	
		public void solve() {
			final Wrap wrap = new Wrap();
			for (int g = 0; g < MAX_GEN; g++) {
				//----------JAVA_8 BEGIN
				wrap.gen = g;
				Arrays.asList(ants).parallelStream().forEach(new Consumer<Ant>() {
				
					public void accept(Ant ant) {
					
						for (int j = 1; j < cityNum; j++) {
							if (!ant.getAllowedCities().isEmpty()) {
								int selectCity = ant.selectNextCity(pheromone);// FIXME
																						// 需要修改的地方
								if (ant.selectCityIsOK(selectCity)) {
									ant.updateLoadQ(selectCity);
								} else {
									if (ant.getTabu().lastElement() != ant.getFirstCity()) {
										ant.getTabu().add(ant.getFirstCity());
									}
									j = j - 1;
									ant.resetLoad();
								}
							}
						}
						if (ant.getTabu().lastElement() != ant.getFirstCity()) {
							ant.getTabu().add(ant.getFirstCity());
						}
						if (ant.getTourLength() < bestLength) {
							bestLength = ant.getTourLength();
							printOptimal(ant, wrap.gen);
						}
						for (int j = 0; j < cityNum; j++) {
							ant.getDelta()[ant.getTabu().get(j).intValue()][ant.getTabu().get(j + 1).intValue()] = (float) (1.0f / ant
									.getTourLength());
							ant.getDelta()[ant.getTabu().get(j + 1).intValue()][ant.getTabu().get(j).intValue()] = (float) (1.0f / ant
									.getTourLength());
						}
					}
				});
				//--------JAVA-8 END
				
				// -------JAVA-7 BEGIN
	//			for (int i = 0; i < antNum; i++) {
	//				// ants[i].resetLoad();
	//				for (int j = 1; j < cityNum; j++) {
	//					if (!ants[i].getAllowedCities().isEmpty()) {
	//						int selectCity = ants[i].selectNextCity(pheromone);// FIXME
	//																			// 需要修改的地方
	//						if (ants[i].selectCityIsOK(selectCity)) {
	//							ants[i].updateLoadQ(selectCity);
	//						} else {
	//							if (ants[i].getTabu().lastElement() != ants[i].getFirstCity()) {
	//								ants[i].getTabu().add(ants[i].getFirstCity());
	//							}
	//							j = j - 1;
	//							ants[i].resetLoad();
	//						}
	//					}
	//				}
	//				if (ants[i].getTabu().lastElement() != ants[i].getFirstCity()) {
	//					ants[i].getTabu().add(ants[i].getFirstCity());
	//				}
	////				print(ants[i],g);
	//				if (ants[i].getTourLength() < bestLength) {
	//					bestLength = ants[i].getTourLength();
	//					printOptimal(ants[i], g);
	//				}
	//				for (int j = 0; j < cityNum; j++) {
	//					ants[i].getDelta()[ants[i].getTabu().get(j).intValue()][ants[i].getTabu().get(j + 1).intValue()] = (float) (1.0f / ants[i]
	//							.getTourLength());
	//					ants[i].getDelta()[ants[i].getTabu().get(j + 1).intValue()][ants[i].getTabu().get(j).intValue()] = (float) (1.0f / ants[i]
	//							.getTourLength());
	//				}
	//			}
				//-------JAVA-7 END
	
				// 更新信息素
				updatePheromone();
	
				// 重新初始化蚂蚁
				for (int i = 0; i < antNum; i++) {
					ants[i].init(distance, alpha, beta, lamda, omga);
				}
			}
	
		}
	
		private void updatePheromone() {
			// 信息素挥发
			for (int i = 0; i < cityNum; i++) {
				for (int j = 0; j < cityNum; j++) {
					pheromone[i][j] = pheromone[i][j] * (1 - rho);
				}
			}
		
			// 信息素更新
			for (int i = 0; i < cityNum; i++) {
				for (int j = 0; j < cityNum; j++) {
					for (int k = 0; k < antNum; k++) {
						pheromone[i][j] += ants[k].getDelta()[i][j];
			
					}
				}
			}
		}

		private void printOptimal(Ant ant, int gen) {
			System.out.println("第" + gen + "代发现新解:" + ant.calculateTourLength() + ",路线为:");
			Vector<Integer> tour = new Vector<Integer>();
			for (Integer m : ant.getTabu()) {
				if (m.intValue() != 0) {
					tour.add(m);
				} else {
					if (tour.size() > 0) {
						System.out.print("0 ");
						for (Integer n : tour) {
							System.out.print(n + " ");
						}
						System.out.println("0");
						tour.clear();
					}
				}
			}
		}

		public static void main(String[] args) throws IOException {
			ACO aco = new ACO(60, 10000, 1.0f, 2.0f, 0.4f, 0.6f, 0.15f);
			aco.init("/Users/jeff/Documents/Eworkspace/AL_DEMOS/src/ant/data.dat");
			aco.solve();
		}

	}


## Note

需要注意的是，蚁群算法并不能每次都取得最优解。