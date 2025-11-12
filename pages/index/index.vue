<template>
	<view class="container">
		<view class="tap" @click="onPage">
			<text>切换计算方式</text>
		</view>
		<view class="card">
			<view class="card-title">赔率输入</view>
			<view class="input-group">
				<view class="input-item">
					<text class="label">主胜赔率</text>
					<input 
						type="number" 
						placeholder="输入主胜赔率" 
						placeholder-class="placeholder"
						v-model="homeOdds"
						@input="validateInput('homeOdds')"
					>
				</view>
				<view class="input-item">
					<text class="label">客胜赔率</text>
					<input 
						type="number" 
						placeholder="输入客胜赔率" 
						placeholder-class="placeholder"
						v-model="awayOdds"
						@input="validateInput('awayOdds')"
					>
				</view>
				<view class="input-item">
					<text class="label">平局赔率</text>
					<input 
						type="number" 
						placeholder="输入平局赔率" 
						placeholder-class="placeholder"
						v-model="drawOdds"
						@input="validateInput('drawOdds')"
					>
				</view>
				<view class="input-item">
					<text class="label">投入本金</text>
					<input 
						type="number" 
						placeholder="输入本金金额" 
						placeholder-class="placeholder"
						v-model="capital"
						@input="validateInput('capital')"
					>
				</view>
			</view>
			
			<view class="quick-actions">
				<text class="quick-title">快捷操作：</text>
				<view class="quick-buttons">
					<button class="quick-btn" @click="clearAll">清空所有</button>
					<button class="quick-btn" @click="fillExample">填充示例</button>
				</view>
			</view>
		</view>

		<!-- 概率分析卡片 -->
		<view class="card" v-if="showProbAnalysis">
			<view class="card-title">概率分析</view>
			<view class="prob-analysis">
				<view class="prob-item">
					<text class="prob-type">主胜概率</text>
					<view class="prob-bar">
						<view class="prob-fill" :style="{width: homeProbPercent + '%'}"></view>
					</view>
					<text class="prob-value">{{ homeProbPercent }}%</text>
				</view>
				<view class="prob-item">
					<text class="prob-type">客胜概率</text>
					<view class="prob-bar">
						<view class="prob-fill" :style="{width: awayProbPercent + '%'}"></view>
					</view>
					<text class="prob-value">{{ awayProbPercent }}%</text>
				</view>
				<view class="prob-item">
					<text class="prob-type">平局概率</text>
					<view class="prob-bar">
						<view class="prob-fill" :style="{width: drawProbPercent + '%'}"></view>
					</view>
					<text class="prob-value">{{ drawProbPercent }}%</text>
				</view>
				<view class="total-prob">
					<text>总隐含概率：{{ totalProbPercent }}%</text>
					<text :class="arbitrageClass">{{ arbitrageMessage }}</text>
				</view>
			</view>
		</view>

		<view class="result-card" v-if="showResult">
			<view class="card-title">
				<text>对冲方案</text>
				<text class="strategy-badge" :class="bestStrategy.strategy">{{ strategyLabels[bestStrategy.strategy] }}</text>
			</view>
			<view class="result-content">
				<template v-if="bestStrategy.strategy === 'arbitrage'">
					<view class="result-type success">
						<text class="icon">💰</text>
						<text>发现套利机会！可无风险盈利 {{ formatCurrency(bestStrategy.minProfit) }}</text>
					</view>
					<view class="bet-details">
						<view class="bet-item">
							<text class="bet-type">主胜下注</text>
							<view class="bet-info">
								<text class="bet-amount">{{ formatCurrency(bestStrategy.homeBet) }}</text>
								<text class="bet-profit success">+{{ formatCurrency(bestStrategy.homeProfit) }}</text>
							</view>
						</view>
						<view class="bet-item">
							<text class="bet-type">客胜下注</text>
							<view class="bet-info">
								<text class="bet-amount">{{ formatCurrency(bestStrategy.awayBet) }}</text>
								<text class="bet-profit success">+{{ formatCurrency(bestStrategy.awayProfit) }}</text>
							</view>
						</view>
						<view class="bet-item">
							<text class="bet-type">平局下注</text>
							<view class="bet-info">
								<text class="bet-amount">{{ formatCurrency(bestStrategy.drawBet) }}</text>
								<text class="bet-profit success">+{{ formatCurrency(bestStrategy.drawProfit) }}</text>
							</view>
						</view>
					</view>
				</template>

				<template v-else-if="bestStrategy.strategy === 'hedge'">
					<view class="result-type warning">
						<text class="icon">🛡️</text>
						<text>对冲方案 - 最小化风险</text>
					</view>
					<view class="bet-details">
						<view class="bet-item">
							<text class="bet-type">主胜下注</text>
							<view class="bet-info">
								<text class="bet-amount">{{ formatCurrency(bestStrategy.homeBet) }}</text>
								<text class="bet-profit" :class="bestStrategy.homeProfit >= 0 ? 'success' : 'danger'">
									{{ bestStrategy.homeProfit >= 0 ? '+' : '' }}{{ formatCurrency(bestStrategy.homeProfit) }}
								</text>
							</view>
						</view>
						<view class="bet-item">
							<text class="bet-type">客胜下注</text>
							<view class="bet-info">
								<text class="bet-amount">{{ formatCurrency(bestStrategy.awayBet) }}</text>
								<text class="bet-profit" :class="bestStrategy.awayProfit >= 0 ? 'success' : 'danger'">
									{{ bestStrategy.awayProfit >= 0 ? '+' : '' }}{{ formatCurrency(bestStrategy.awayProfit) }}
								</text>
							</view>
						</view>
						<view class="bet-item">
							<text class="bet-type">平局下注</text>
							<view class="bet-info">
								<text class="bet-amount">{{ formatCurrency(bestStrategy.drawBet) }}</text>
								<text class="bet-profit" :class="bestStrategy.drawProfit >= 0 ? 'success' : 'danger'">
									{{ bestStrategy.drawProfit >= 0 ? '+' : '' }}{{ formatCurrency(bestStrategy.drawProfit) }}
								</text>
							</view>
						</view>
					</view>
				</template>

				<template v-else>
					<view class="result-type danger">
						<text class="icon">❌</text>
						<text>无法找到有效对冲方案</text>
					</view>
					<view class="suggestion">
						<text>建议：调整赔率输入或增加本金</text>
					</view>
				</template>

				<view class="summary" v-if="bestStrategy.strategy !== 'none'">
					<view class="summary-item">
						<text class="summary-label">总下注金额</text>
						<text class="summary-value">{{ formatCurrency(bestStrategy.totalBet) }}</text>
					</view>
					<view class="summary-item">
						<text class="summary-label">剩余资金</text>
						<text class="summary-value">{{ formatCurrency(capital - bestStrategy.totalBet) }}</text>
					</view>
					<view class="summary-item" :class="bestStrategy.minProfit >= 0 ? 'success' : 'danger'">
						<text class="summary-label">最低保证{{ bestStrategy.minProfit >= 0 ? '盈利' : '亏损' }}</text>
						<text class="summary-value">
							{{ bestStrategy.minProfit >= 0 ? '+' : '' }}{{ formatCurrency(bestStrategy.minProfit) }}
						</text>
					</view>
					<view class="summary-item" v-if="bestStrategy.roi">
						<text class="summary-label">投资回报率</text>
						<text class="summary-value" :class="bestStrategy.roi >= 0 ? 'success' : 'danger'">
							{{ (bestStrategy.roi * 100).toFixed(2) }}%
						</text>
					</view>
				</view>
			</view>
		</view>

		<view class="action-bar">
			<button class="calculate-btn" @click="calculate" :disabled="!isFormValid">
				<text v-if="calculating" class="loading">计算中...</text>
				<text v-else>计算最优对冲方案</text>
			</button>
		</view>

		<view class="tips-card">
			<view class="card-title">使用说明</view>
			<view class="tips-content">
				<text>• <text class="tip-highlight">套利机会</text>：当总隐含概率＜100%时出现，可无风险盈利</text>
				<text>• <text class="tip-highlight">对冲方案</text>：当总隐含概率≥100%时，最小化风险的最优分配</text>
				<text>• 绿色表示盈利，红色表示亏损，黄色表示对冲保护</text>
				<text>• 投资有风险，请谨慎决策，建议小额试水</text>
			</view>
		</view>
	</view>
</template>

<script setup>
import { ref, computed, watch } from 'vue';

const homeOdds = ref('');
const awayOdds = ref('');
const drawOdds = ref('');
const capital = ref('');
const showResult = ref(false);
const showProbAnalysis = ref(false);
const calculating = ref(false);
const bestStrategy = ref({});

const strategyLabels = {
	'arbitrage': '套利机会',
	'hedge': '对冲方案',
	'none': '无方案'
};

const onPage = () =>{
	uni.navigateTo({
		url: '/pages/home/home'
	})
}

// 计算概率分析
const homeProb = computed(() => homeOdds.value ? 1 / parseFloat(homeOdds.value) : 0);
const awayProb = computed(() => awayOdds.value ? 1 / parseFloat(awayOdds.value) : 0);
const drawProb = computed(() => drawOdds.value ? 1 / parseFloat(drawOdds.value) : 0);
const totalProb = computed(() => homeProb.value + awayProb.value + drawProb.value);

const homeProbPercent = computed(() => (homeProb.value * 100).toFixed(1));
const awayProbPercent = computed(() => (awayProb.value * 100).toFixed(1));
const drawProbPercent = computed(() => (drawProb.value * 100).toFixed(1));
const totalProbPercent = computed(() => (totalProb.value * 100).toFixed(1));

const arbitrageClass = computed(() => {
	if (totalProb.value < 0.98) return 'success';
	if (totalProb.value < 1.02) return 'warning';
	return 'danger';
});

const arbitrageMessage = computed(() => {
	if (totalProb.value < 0.98) return `套利空间：${(100 - totalProb.value * 100).toFixed(2)}%`;
	if (totalProb.value < 1.02) return '接近公平赔率';
	return '庄家优势明显';
});

// 计算表单是否有效
const isFormValid = computed(() => {
	return homeOdds.value > 0 && 
	       awayOdds.value > 0 && 
	       drawOdds.value > 0 && 
	       capital.value > 0;
});

// 监听赔率变化，自动显示概率分析
watch([homeOdds, awayOdds, drawOdds], () => {
	if (homeOdds.value && awayOdds.value && drawOdds.value) {
		showProbAnalysis.value = true;
	}
});

// 输入验证
function validateInput(field) {
	const value = parseFloat(this[field]);
	if (value <= 0) {
		this[field] = '';
	}
}

// 格式化货币显示
function formatCurrency(amount) {
	return `¥${parseFloat(amount).toFixed(2)}`;
}

// 清空所有输入
function clearAll() {
	homeOdds.value = '';
	awayOdds.value = '';
	drawOdds.value = '';
	capital.value = '';
	showResult.value = false;
	showProbAnalysis.value = false;
}

// 填充示例数据
function fillExample() {
	homeOdds.value = '2.5';
	awayOdds.value = '3.2';
	drawOdds.value = '3.0';
	capital.value = '1000';
	showProbAnalysis.value = true;
}

async function calculate() {
	if (!isFormValid.value) {
		uni.showToast({ 
			title: '请填写所有有效字段', 
			icon: 'none',
			duration: 2000
		});
		return;
	}

	calculating.value = true;
	
	// 模拟计算延迟，让用户看到加载状态
	await new Promise(resolve => setTimeout(resolve, 500));

	try {
		const home = parseFloat(homeOdds.value);
		const away = parseFloat(awayOdds.value);
		const draw = parseFloat(drawOdds.value);
		const totalCapital = parseFloat(capital.value);

		// 检查套利机会（总概率 < 1）
		if (totalProb.value < 0.98) {
			calculateArbitrage(totalCapital, home, away, draw);
		} else {
			calculateHedge(totalCapital, home, away, draw);
		}

		showResult.value = true;
	} catch (error) {
		console.error('计算错误:', error);
		uni.showToast({ 
			title: '计算失败，请检查输入', 
			icon: 'none',
			duration: 2000
		});
	} finally {
		calculating.value = false;
	}
}

function calculateArbitrage(totalCapital, home, away, draw) {
	// 套利计算 - 按概率比例分配资金
	const homeBet = (homeProb.value / totalProb.value) * totalCapital;
	const awayBet = (awayProb.value / totalProb.value) * totalCapital;
	const drawBet = (drawProb.value / totalProb.value) * totalCapital;

	const homeProfit = homeBet * home - totalCapital;
	const awayProfit = awayBet * away - totalCapital;
	const drawProfit = drawBet * draw - totalCapital;
	const minProfit = Math.min(homeProfit, awayProfit, drawProfit);
	const roi = minProfit / totalCapital;

	bestStrategy.value = {
		strategy: 'arbitrage',
		homeBet,
		awayBet,
		drawBet,
		totalBet: homeBet + awayBet + drawBet,
		homeProfit,
		awayProfit,
		drawProfit,
		minProfit,
		roi
	};
}

function calculateHedge(totalCapital, home, away, draw) {
	let bestSolution = null;
	let bestMinProfit = -Infinity;

	// 使用更精确的优化算法
	const steps = 100; // 增加精度
	const minRatio = 0.1; // 最小下注比例
	const maxRatio = 0.8; // 最大下注比例

	for (let i = 1; i <= steps; i++) {
		const homeRatio = minRatio + (maxRatio - minRatio) * (i / steps);
		const homeBet = totalCapital * homeRatio;
		
		const remainingCapital = totalCapital - homeBet;
		
		// 优化剩余资金的分配
		for (let j = 1; j <= steps; j++) {
			const awayRatio = minRatio + (maxRatio - minRatio) * (j / steps);
			const awayBet = Math.min(remainingCapital * awayRatio, remainingCapital * 0.9);
			const drawBet = remainingCapital - awayBet;

			if (awayBet <= 0 || drawBet <= 0 || awayBet > remainingCapital || drawBet > remainingCapital) continue;

			const homeProfit = homeBet * home - totalCapital;
			const awayProfit = awayBet * away - totalCapital;
			const drawProfit = drawBet * draw - totalCapital;

			const minProfit = Math.min(homeProfit, awayProfit, drawProfit);

			// 优先选择所有结果都盈利的方案
			if (homeProfit > 0 && awayProfit > 0 && drawProfit > 0) {
				if (minProfit > bestMinProfit) {
					bestMinProfit = minProfit;
					bestSolution = {
						homeBet,
						awayBet,
						drawBet,
						homeProfit,
						awayProfit,
						drawProfit,
						minProfit
					};
				}
			} else if (!bestSolution && minProfit > bestMinProfit) {
				// 如果没有全盈利方案，选择最优的
				bestMinProfit = minProfit;
				bestSolution = {
					homeBet,
					awayBet,
					drawBet,
					homeProfit,
					awayProfit,
					drawProfit,
					minProfit
				};
			}
		}
	}

	if (bestSolution) {
		const roi = bestSolution.minProfit / totalCapital;
		bestStrategy.value = {
			strategy: 'hedge',
			...bestSolution,
			totalBet: bestSolution.homeBet + bestSolution.awayBet + bestSolution.drawBet,
			roi
		};
	} else {
		bestStrategy.value = { strategy: 'none' };
	}
}
</script>

<style scoped lang="scss">
.container {
	padding: 24rpx;
	background: #f5f6fa;
	min-height: 100vh;
}

.tap{
	margin: 20rpx 0;
	display: flex;
	>text{
		padding: 10rpx 30rpx;
		box-sizing: border-box;
		background-color: #10b981;
		color: #fff;
		border-radius: 20rpx;
	}
}

.card {
	background: white;
	border-radius: 16rpx;
	padding: 32rpx;
	margin-bottom: 24rpx;
	box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.08);
}

.card-title {
	display: flex;
	justify-content: space-between;
	align-items: center;
	font-size: 32rpx;
	font-weight: 600;
	color: #333;
	margin-bottom: 24rpx;
	padding-bottom: 16rpx;
	border-bottom: 1rpx solid #eee;
}

.strategy-badge {
	font-size: 24rpx;
	padding: 8rpx 16rpx;
	border-radius: 20rpx;
	font-weight: 500;
	
	&.arbitrage {
		background: #dcfce7;
		color: #16a34a;
	}
	
	&.hedge {
		background: #fef3c7;
		color: #d97706;
	}
	
	&.none {
		background: #fecaca;
		color: #dc2626;
	}
}

.input-group {
	display: flex;
	flex-direction: column;
	gap: 24rpx;
}

.input-item {
	display: flex;
	align-items: center;
	justify-content: space-between;
	
	.label {
		font-size: 28rpx;
		color: #333;
		font-weight: 500;
		width: 180rpx;
	}
	
	input {
		flex: 1;
		border: 1rpx solid #dcdfe6;
		border-radius: 12rpx;
		padding: 20rpx 24rpx;
		font-size: 28rpx;
		background: #fafbfc;
		
		&:focus {
			border-color: #409eff;
			background: white;
		}
	}
}

.quick-actions {
	margin-top: 24rpx;
	padding-top: 24rpx;
	border-top: 1rpx solid #eee;
}

.quick-title {
	font-size: 28rpx;
	color: #666;
	margin-bottom: 16rpx;
	display: block;
}

.quick-buttons {
	display: flex;
	gap: 16rpx;
}

.quick-btn {
	flex: 1;
	background: #f1f5f9;
	color: #475569;
	border: none;
	border-radius: 8rpx;
	padding: 16rpx;
	font-size: 26rpx;
	
	&:active {
		background: #e2e8f0;
	}
}

.placeholder {
	color: #c0c4cc;
	font-size: 28rpx;
}

// 概率分析样式
.prob-analysis {
	display: flex;
	flex-direction: column;
	gap: 20rpx;
}

.prob-item {
	display: flex;
	align-items: center;
	gap: 20rpx;
}

.prob-type {
	font-size: 26rpx;
	color: #666;
	width: 120rpx;
}

.prob-bar {
	flex: 1;
	height: 20rpx;
	background: #e2e8f0;
	border-radius: 10rpx;
	overflow: hidden;
}

.prob-fill {
	height: 100%;
	background: linear-gradient(90deg, #3b82f6, #06b6d4);
	border-radius: 10rpx;
	transition: width 0.3s ease;
}

.prob-value {
	font-size: 26rpx;
	font-weight: 600;
	color: #333;
	width: 80rpx;
	text-align: right;
}

.total-prob {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-top: 16rpx;
	padding-top: 16rpx;
	border-top: 1rpx solid #e2e8f0;
	font-weight: 600;
	
	.success {
		color: #10b981;
	}
	
	.warning {
		color: #f59e0b;
	}
	
	.danger {
		color: #ef4444;
	}
}

.result-card {
	background: white;
	border-radius: 16rpx;
	padding: 32rpx;
	margin-bottom: 24rpx;
	box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.08);
}

.result-content {
	display: flex;
	flex-direction: column;
	gap: 24rpx;
}

.result-type {
	display: flex;
	align-items: center;
	gap: 16rpx;
	padding: 20rpx;
	border-radius: 12rpx;
	font-weight: 500;
	
	&.success {
		background: #f0f9ff;
		color: #06b6d4;
	}
	
	&.warning {
		background: #fffbf0;
		color: #f59e0b;
	}
	
	&.danger {
		background: #fef2f2;
		color: #ef4444;
	}
	
	.icon {
		font-size: 36rpx;
	}
}

.bet-details {
	display: flex;
	flex-direction: column;
	gap: 16rpx;
}

.bet-item {
	display: flex;
	justify-content: space-between;
	align-items: center;
	padding: 20rpx;
	background: #f8fafc;
	border-radius: 12rpx;
}

.bet-type {
	font-size: 28rpx;
	color: #666;
	width: 160rpx;
}

.bet-info {
	display: flex;
	flex-direction: column;
	align-items: flex-end;
	gap: 4rpx;
}

.bet-amount {
	font-size: 28rpx;
	font-weight: 600;
	color: #333;
}

.bet-profit {
	font-size: 24rpx;
	font-weight: 500;
	
	&.success {
		color: #10b981;
	}
	
	&.danger {
		color: #ef4444;
	}
}

.suggestion {
	padding: 20rpx;
	background: #f8fafc;
	border-radius: 12rpx;
	color: #666;
	font-size: 28rpx;
	text-align: center;
}

.summary {
	display: flex;
	flex-direction: column;
	gap: 16rpx;
	padding: 24rpx;
	background: #f8fafc;
	border-radius: 12rpx;
	border-top: 1rpx solid #e2e8f0;
}

.summary-item {
	display: flex;
	justify-content: space-between;
	align-items: center;
	
	.summary-label {
		font-size: 28rpx;
		color: #666;
	}
	
	.summary-value {
		font-size: 28rpx;
		font-weight: 600;
		
		&.success {
			color: #10b981;
		}
		
		&.danger {
			color: #ef4444;
		}
	}
	
	&.success .summary-value {
		color: #10b981;
	}
	
	&.danger .summary-value {
		color: #ef4444;
	}
}

.action-bar {
	padding: 24rpx 0;
}

.calculate-btn {
	width: 100%;
	background: #3b82f6;
	color: white;
	border: none;
	border-radius: 12rpx;
	padding: 24rpx;
	font-size: 32rpx;
	font-weight: 500;
	position: relative;
	
	&:active {
		background: #2563eb;
	}
	
	&:disabled {
		background: #9ca3af;
		opacity: 0.6;
	}
	
	.loading {
		display: inline-flex;
		align-items: center;
		gap: 8rpx;
		
		&::after {
			content: '';
			width: 16rpx;
			height: 16rpx;
			border: 2rpx solid transparent;
			border-top: 2rpx solid white;
			border-radius: 50%;
			animation: spin 1s linear infinite;
		}
	}
}

@keyframes spin {
	0% { transform: rotate(0deg); }
	100% { transform: rotate(360deg); }
}

.tips-card {
	background: white;
	border-radius: 16rpx;
	padding: 32rpx;
	box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.08);
}

.tips-content {
	display: flex;
	flex-direction: column;
	gap: 16rpx;
	
	text {
		font-size: 26rpx;
		color: #666;
		line-height: 1.6;
	}
}

.tip-highlight {
	color: #3b82f6;
	font-weight: 500;
}
</style>