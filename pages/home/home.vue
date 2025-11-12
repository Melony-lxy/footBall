<template>
	<view class="container">
		<view class="header">
			<text class="title"></text>
			<text class="subtitle" @click="onPage">切换计算方式</text>
		</view>

		<view class="card">
			<view class="card-title">
				<text>赔率输入</text>
				<view class="odds-tips">
					<text class="tip-dot"></text>
					<text class="tip-text">赔率需大于1</text>
				</view>
			</view>
			<view class="input-group">
				<view class="input-item">
					<view class="input-label">
						<text class="label-text">主胜赔率</text>
						<text class="label-odds">1赔{{ homeOdds || '?' }}</text>
					</view>
					<input 
						type="number" 
						placeholder="例如：2.5" 
						placeholder-class="placeholder"
						v-model="homeOdds"
						@input="handleInputChange"
						:class="{ 'input-error': homeOdds && homeOdds <= 1 }"
					>
					<view class="input-hint" v-if="homeOdds && homeOdds > 1">
						返还率：{{ (100/homeOdds).toFixed(1) }}%
					</view>
				</view>

				<view class="input-item">
					<view class="input-label">
						<text class="label-text">客胜赔率</text>
						<text class="label-odds">1赔{{ awayOdds || '?' }}</text>
					</view>
					<input 
						type="number" 
						placeholder="例如：1.8" 
						placeholder-class="placeholder"
						v-model="awayOdds"
						@input="handleInputChange"
						:class="{ 'input-error': awayOdds && awayOdds <= 1 }"
					>
					<view class="input-hint" v-if="awayOdds && awayOdds > 1">
						返还率：{{ (100/awayOdds).toFixed(1) }}%
					</view>
				</view>

				<view class="input-item">
					<view class="input-label">
						<text class="label-text">投入本金</text>
						<text class="label-odds">{{ formatCurrency(capital) }}</text>
					</view>
					<input 
						type="number" 
						placeholder="例如：1000" 
						placeholder-class="placeholder"
						v-model="capital"
						@input="handleInputChange"
						:class="{ 'input-error': capital && capital <= 0 }"
					>
					<view class="input-hint">
						建议金额：100-10000元
					</view>
				</view>
			</view>

			<view class="quick-actions">
				<view class="quick-title">快捷操作：</view>
				<view class="quick-buttons">
					<button class="quick-btn" @click="clearAll" :disabled="!hasInput">
						<text class="btn-icon">🗑️</text>
						<text>清空所有</text>
					</button>
					<button class="quick-btn" @click="fillExample">
						<text class="btn-icon">📋</text>
						<text>填充示例</text>
					</button>
				</view>
			</view>
		</view>

		<!-- 概率分析 -->
		<view class="card probability-card" v-if="showProbability">
			<view class="card-title">概率分析</view>
			<view class="probability-content">
				<view class="prob-item">
					<text class="prob-label">主胜概率</text>
					<view class="prob-bar">
						<view class="prob-fill home-fill" :style="{ width: homeProbPercent + '%' }"></view>
					</view>
					<text class="prob-value">{{ homeProbPercent }}%</text>
				</view>
				<view class="prob-item">
					<text class="prob-label">客胜概率</text>
					<view class="prob-bar">
						<view class="prob-fill away-fill" :style="{ width: awayProbPercent + '%' }"></view>
					</view>
					<text class="prob-value">{{ awayProbPercent }}%</text>
				</view>
				<view class="total-prob" :class="arbitrageClass">
					<text class="total-label">总隐含概率</text>
					<text class="total-value">{{ totalProbPercent }}%</text>
					<text class="arbitrage-tag">{{ arbitrageMessage }}</text>
				</view>
			</view>
		</view>

		<!-- 结果展示 -->
		<view class="result-card" v-if="showResult && !errorMessage">
			<view class="card-title result-title">
				<text>最优对冲方案</text>
				<view class="result-badge" :class="arbitrageClass">
					{{ arbitrageTag }}
				</view>
			</view>
			
			<view class="betting-plan">
				<view class="plan-item">
					<view class="plan-icon home-icon">🏠</view>
					<view class="plan-details">
						<text class="plan-label">主胜投注</text>
						<text class="plan-amount">{{ homeBet.toFixed(2) }} 元</text>
						<text class="plan-odds">赔率 {{ homeOdds }}</text>
					</view>
					<view class="plan-profit" :class="getProfitClass(homeProfit)">
						{{ homeProfit >= 0 ? '+' : '' }}{{ homeProfit.toFixed(2) }}元
					</view>
				</view>

				<view class="plan-item">
					<view class="plan-icon away-icon">✈️</view>
					<view class="plan-details">
						<text class="plan-label">客胜投注</text>
						<text class="plan-amount">{{ awayBet.toFixed(2) }} 元</text>
						<text class="plan-odds">赔率 {{ awayOdds }}</text>
					</view>
					<view class="plan-profit" :class="getProfitClass(awayProfit)">
						{{ awayProfit >= 0 ? '+' : '' }}{{ awayProfit.toFixed(2) }}元
					</view>
				</view>
			</view>

			<view class="result-summary">
				<view class="summary-item">
					<text class="summary-label">总投注金额</text>
					<text class="summary-value">{{ totalBet.toFixed(2) }} 元</text>
				</view>
				<view class="summary-item">
					<text class="summary-label">剩余资金</text>
					<text class="summary-value">{{ (capital - totalBet).toFixed(2) }} 元</text>
				</view>
				<view class="summary-item highlight" :class="getProfitClass(profit)">
					<text class="summary-label">保证盈利</text>
					<text class="summary-value">
						{{ profit >= 0 ? '+' : '' }}{{ profit.toFixed(2) }} 元
					</text>
				</view>
				<view class="summary-item highlight" :class="getProfitClass(profit)">
					<text class="summary-label">收益率</text>
					<text class="summary-value">{{ (yieldRate * 100).toFixed(2) }}%</text>
				</view>
			</view>

			<view class="risk-notice" v-if="profit < 0">
				<text class="notice-icon">⚠️</text>
				<text class="notice-text">此为对冲方案，不能保证盈利，但能有效降低风险</text>
			</view>
		</view>

		<!-- 错误提示 -->
		<view class="error-card" v-if="errorMessage">
			<view class="error-header">
				<text class="error-icon">❌</text>
				<text class="error-title">无法计算</text>
			</view>
			<text class="error-message">{{ errorMessage }}</text>
			<view class="error-suggestion" v-if="errorSuggestion">
				<text class="suggestion-title">建议：</text>
				<text>{{ errorSuggestion }}</text>
			</view>
		</view>

		<!-- 使用说明 -->
		<view class="tips-card">
			<view class="card-title">使用说明</view>
			<view class="tips-content">
				<view class="tip-item">
					<text class="tip-icon">💡</text>
					<text class="tip-text">套利条件：1/主胜赔率 + 1/客胜赔率 < 1</text>
				</view>
				<view class="tip-item">
					<text class="tip-icon">🛡️</text>
					<text class="tip-text">对冲方案可降低风险，但不保证盈利</text>
				</view>
				<view class="tip-item">
					<text class="tip-icon">💰</text>
					<text class="tip-text">绿色数字表示盈利，红色表示亏损</text>
				</view>
				<view class="tip-item">
					<text class="tip-icon">⚠️</text>
					<text class="tip-text">投资有风险，请理性投注</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script setup>
import { ref, computed, watch } from 'vue';

// 输入数据
const homeOdds = ref('');
const awayOdds = ref('');
const capital = ref('');

// 计算结果
const homeBet = ref(0);
const awayBet = ref(0);
const homeProfit = ref(0);
const awayProfit = ref(0);
const profit = ref(0);
const yieldRate = ref(0);
const showResult = ref(false);
const showProbability = ref(false);
const errorMessage = ref('');
const errorSuggestion = ref('');

// 计算属性
const hasInput = computed(() => homeOdds.value || awayOdds.value || capital.value);

const homeProb = computed(() => homeOdds.value ? 1 / parseFloat(homeOdds.value) : 0);
const awayProb = computed(() => awayOdds.value ? 1 / parseFloat(awayOdds.value) : 0);
const totalProb = computed(() => homeProb.value + awayProb.value);

const homeProbPercent = computed(() => (homeProb.value * 100).toFixed(1));
const awayProbPercent = computed(() => (awayProb.value * 100).toFixed(1));
const totalProbPercent = computed(() => (totalProb.value * 100).toFixed(1));

const onPage = () =>{
	uni.navigateBack()
}

const arbitrageClass = computed(() => {
	if (totalProb.value < 0.98) return 'arbitrage';
	if (totalProb.value < 1.02) return 'neutral';
	return 'no-arbitrage';
});

const arbitrageMessage = computed(() => {
	if (totalProb.value < 0.98) return `套利空间 ${(100 - totalProb.value * 100).toFixed(1)}%`;
	if (totalProb.value < 1.02) return '接近公平';
	return '无套利空间';
});

const arbitrageTag = computed(() => {
	if (totalProb.value < 0.98) return '套利机会';
	if (totalProb.value < 1.02) return '对冲方案';
	return '风险对冲';
});

const totalBet = computed(() => homeBet.value + awayBet.value);

// 处理方法
const handleInputChange = () => {
	// 延迟计算，避免频繁计算
	clearTimeout(handleInputChange.timer);
	handleInputChange.timer = setTimeout(calculate, 300);
};

const formatCurrency = (amount) => {
	if (!amount) return '0元';
	return `${parseFloat(amount).toFixed(2)}元`;
};

const getProfitClass = (profitValue) => {
	return profitValue >= 0 ? 'profit-positive' : 'profit-negative';
};

const calculate = () => {
	// 重置状态
	showResult.value = false;
	errorMessage.value = '';
	errorSuggestion.value = '';
	showProbability.value = false;

	// 验证输入
	if (!homeOdds.value || !awayOdds.value || !capital.value) {
		showProbability.value = homeOdds.value && awayOdds.value;
		return;
	}

	const home = parseFloat(homeOdds.value);
	const away = parseFloat(awayOdds.value);
	const totalCapital = parseFloat(capital.value);

	// 输入验证
	if (home <= 1 || away <= 1) {
		errorMessage.value = '赔率必须大于1';
		errorSuggestion.value = '请检查赔率输入是否正确';
		return;
	}

	if (totalCapital <= 0) {
		errorMessage.value = '本金必须大于0';
		errorSuggestion.value = '请输入有效的本金金额';
		return;
	}

	if (totalCapital < 10) {
		errorMessage.value = '本金过少';
		errorSuggestion.value = '建议本金至少10元';
		return;
	}

	showProbability.value = true;

	// 检查套利条件
	const arbitrageCondition = totalProb.value;
	
	if (arbitrageCondition >= 1) {
		// 无套利空间，计算对冲方案
		calculateHedge(totalCapital, home, away);
	} else {
		// 有套利空间，计算套利方案
		calculateArbitrage(totalCapital, home, away);
	}

	showResult.value = true;
};

const calculateArbitrage = (totalCapital, home, away) => {
	// 套利计算 - 按概率比例分配
	const ratio = homeProb.value / totalProb.value;
	homeBet.value = totalCapital * ratio;
	awayBet.value = totalCapital * (1 - ratio);

	// 计算盈利
	homeProfit.value = homeBet.value * home - totalCapital;
	awayProfit.value = awayBet.value * away - totalCapital;
	profit.value = Math.min(homeProfit.value, awayProfit.value);
	yieldRate.value = profit.value / totalCapital;
};

const calculateHedge = (totalCapital, home, away) => {
	// 对冲计算 - 寻找最优分配
	let bestRatio = 0.5;
	let bestProfit = -Infinity;

	// 在0.1到0.9之间寻找最优比例
	for (let ratio = 0.1; ratio <= 0.9; ratio += 0.01) {
		const homeBetTemp = totalCapital * ratio;
		const awayBetTemp = totalCapital * (1 - ratio);
		
		const homeProfitTemp = homeBetTemp * home - totalCapital;
		const awayProfitTemp = awayBetTemp * away - totalCapital;
		const minProfit = Math.min(homeProfitTemp, awayProfitTemp);

		if (minProfit > bestProfit) {
			bestProfit = minProfit;
			bestRatio = ratio;
		}
	}

	homeBet.value = totalCapital * bestRatio;
	awayBet.value = totalCapital * (1 - bestRatio);
	homeProfit.value = homeBet.value * home - totalCapital;
	awayProfit.value = awayBet.value * away - totalCapital;
	profit.value = bestProfit;
	yieldRate.value = profit.value / totalCapital;
};

const clearAll = () => {
	homeOdds.value = '';
	awayOdds.value = '';
	capital.value = '';
	showResult.value = false;
	showProbability.value = false;
	errorMessage.value = '';
};

const fillExample = () => {
	homeOdds.value = '3.0';
	awayOdds.value = '2.0';
	capital.value = '1000';
	calculate();
};
</script>

<style scoped lang="scss">
.container {
	padding: 24rpx;
	background: linear-gradient(135deg, #667eea 0%, #9c38ad 100%);
	min-height: 100vh;
}

.header {
	text-align: center;
	margin-bottom: 32rpx;
	
	.title {
		display: block;
		font-size: 36rpx;
		font-weight: bold;
		color: white;
		margin-bottom: 8rpx;
	}
	
	.subtitle {

		color: rgba(255, 255, 255, 0.8);
		padding: 10rpx 30rpx;
		box-sizing: border-box;
		background-color: #10b981;
		color: #fff;
		border-radius: 20rpx;
	}
}

.card {
	background: white;
	border-radius: 20rpx;
	padding: 32rpx;
	margin-bottom: 24rpx;
	box-shadow: 0 8rpx 32rpx rgba(0, 0, 0, 0.1);
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
	border-bottom: 1rpx solid #f0f0f0;
}

.odds-tips {
	display: flex;
	align-items: center;
	gap: 8rpx;
	
	.tip-dot {
		width: 8rpx;
		height: 8rpx;
		background: #ff6b6b;
		border-radius: 50%;
	}
	
	.tip-text {
		font-size: 24rpx;
		color: #666;
	}
}

.input-group {
	display: flex;
	flex-direction: column;
	gap: 32rpx;
}

.input-item {
	.input-label {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 16rpx;
		
		.label-text {
			font-size: 28rpx;
			color: #333;
			font-weight: 500;
		}
		
		.label-odds {
			font-size: 24rpx;
			color: #666;
		}
	}
	
	input {
		// width: 100%;
		border: 2rpx solid #e1e5e9;
		border-radius: 12rpx;
		padding: 24rpx;
		font-size: 28rpx;
		background: #f8f9fa;
		transition: all 0.3s ease;
		
		&:focus {
			border-color: #667eea;
			background: white;
			box-shadow: 0 0 0 3rpx rgba(102, 126, 234, 0.1);
		}
		
		&.input-error {
			border-color: #ff6b6b;
			background: #fff5f5;
		}
	}
	
	.input-hint {
		font-size: 24rpx;
		color: #888;
		margin-top: 8rpx;
	}
}

.placeholder {
	color: #c0c4cc;
	font-size: 28rpx;
}

.quick-actions {
	margin-top: 32rpx;
	padding-top: 24rpx;
	border-top: 1rpx solid #f0f0f0;
}

.quick-title {
	font-size: 28rpx;
	color: #666;
	margin-bottom: 16rpx;
}

.quick-buttons {
	display: flex;
	gap: 16rpx;
}

.quick-btn {
	flex: 1;
	display: flex;
	align-items: center;
	justify-content: center;
	gap: 8rpx;
	background: #f8f9fa;
	color: #666;
	border: none;
	border-radius: 12rpx;
	padding: 20rpx;
	font-size: 26rpx;
	
	&:active {
		background: #e9ecef;
	}
	
	&:disabled {
		opacity: 0.5;
	}
	
	.btn-icon {
		font-size: 28rpx;
	}
}

// 概率分析样式
.probability-card {
	.probability-content {
		display: flex;
		flex-direction: column;
		gap: 24rpx;
	}
	
	.prob-item {
		display: flex;
		align-items: center;
		gap: 20rpx;
	}
	
	.prob-label {
		font-size: 26rpx;
		color: #666;
		width: 140rpx;
	}
	
	.prob-bar {
		flex: 1;
		height: 16rpx;
		background: #e9ecef;
		border-radius: 8rpx;
		overflow: hidden;
	}
	
	.prob-fill {
		height: 100%;
		border-radius: 8rpx;
		transition: width 0.5s ease;
		
		&.home-fill {
			background: linear-gradient(90deg, #667eea, #764ba2);
		}
		
		&.away-fill {
			background: linear-gradient(90deg, #f093fb, #f5576c);
		}
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
		padding: 20rpx;
		border-radius: 12rpx;
		font-weight: 600;
		
		&.arbitrage {
			background: #e3f2fd;
			color: #1976d2;
		}
		
		&.neutral {
			background: #fff3e0;
			color: #f57c00;
		}
		
		&.no-arbitrage {
			background: #ffebee;
			color: #d32f2f;
		}
		
		.arbitrage-tag {
			font-size: 24rpx;
			font-weight: 500;
		}
	}
}

// 结果卡片样式
.result-card {
	.result-title {
		display: flex;
		justify-content: space-between;
		align-items: center;
	}
	
	.result-badge {
		padding: 8rpx 16rpx;
		border-radius: 20rpx;
		font-size: 24rpx;
		font-weight: 500;
		
		&.arbitrage {
			background: #e3f2fd;
			color: #1976d2;
		}
		
		&.neutral {
			background: #fff3e0;
			color: #f57c00;
		}
		
		&.no-arbitrage {
			background: #ffebee;
			color: #d32f2f;
		}
	}
}

.betting-plan {
	display: flex;
	flex-direction: column;
	gap: 20rpx;
	margin-bottom: 32rpx;
}

.plan-item {
	display: flex;
	align-items: center;
	gap: 20rpx;
	padding: 24rpx;
	background: #f8f9fa;
	border-radius: 16rpx;
	
	.plan-icon {
		font-size: 40rpx;
		width: 60rpx;
		text-align: center;
	}
	
	.plan-details {
		flex: 1;
		display: flex;
		flex-direction: column;
		gap: 4rpx;
	}
	
	.plan-label {
		font-size: 26rpx;
		color: #666;
	}
	
	.plan-amount {
		font-size: 28rpx;
		font-weight: 600;
		color: #333;
	}
	
	.plan-odds {
		font-size: 24rpx;
		color: #888;
	}
	
	.plan-profit {
		font-size: 28rpx;
		font-weight: 600;
		padding: 8rpx 16rpx;
		border-radius: 8rpx;
		
		&.profit-positive {
			background: #e8f5e8;
			color: #2e7d32;
		}
		
		&.profit-negative {
			background: #ffebee;
			color: #d32f2f;
		}
	}
}

.result-summary {
	display: flex;
	flex-direction: column;
	gap: 16rpx;
	padding: 24rpx;
	background: #f8f9fa;
	border-radius: 16rpx;
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
		color: #333;
	}
	
	&.highlight {
		padding-top: 16rpx;
		border-top: 1rpx solid #e1e5e9;
		
		.summary-label {
			font-size: 30rpx;
			font-weight: 600;
		}
		
		.summary-value {
			font-size: 32rpx;
		}
		
		&.profit-positive .summary-value {
			color: #2e7d32;
		}
		
		&.profit-negative .summary-value {
			color: #d32f2f;
		}
	}
}

.risk-notice {
	display: flex;
	align-items: center;
	gap: 12rpx;
	padding: 20rpx;
	background: #fff3e0;
	border-radius: 12rpx;
	margin-top: 24rpx;
	
	.notice-icon {
		font-size: 28rpx;
	}
	
	.notice-text {
		font-size: 26rpx;
		color: #f57c00;
		flex: 1;
	}
}

// 错误卡片样式
.error-card {
	background: #fff5f5;
	border: 1rpx solid #ffcdd2;
	border-radius: 16rpx;
	padding: 32rpx;
	margin-bottom: 24rpx;
	
	.error-header {
		display: flex;
		align-items: center;
		gap: 12rpx;
		margin-bottom: 16rpx;
	}
	
	.error-icon {
		font-size: 32rpx;
	}
	
	.error-title {
		font-size: 28rpx;
		font-weight: 600;
		color: #d32f2f;
	}
	
	.error-message {
		font-size: 28rpx;
		color: #d32f2f;
		margin-bottom: 16rpx;
		display: block;
	}
	
	.error-suggestion {
		font-size: 26rpx;
		color: #666;
		
		.suggestion-title {
			font-weight: 600;
			color: #333;
		}
	}
}

// 使用说明样式
.tips-card {
	.tips-content {
		display: flex;
		flex-direction: column;
		gap: 20rpx;
	}
	
	.tip-item {
		display: flex;
		align-items: flex-start;
		gap: 16rpx;
	}
	
	.tip-icon {
		font-size: 28rpx;
		width: 40rpx;
		text-align: center;
	}
	
	.tip-text {
		font-size: 26rpx;
		color: #fff;
		line-height: 1.5;
		flex: 1;
	}
}
</style>