<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>电商回本测算工具</title>
    <style>
        /* 这里是 CSS（网页外观） */
        body {
            font-family: 'PingFang SC', 'Microsoft YaHei', sans-serif;
            background-color: #f5f7fa;
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
        }
        .container {
            background-color: #ffffff;
            width: 100%;
            max-width: 400px;
            padding: 25px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.05);
        }
        h2 {
            text-align: center;
            color: #333;
            margin-top: 0;
            margin-bottom: 20px;
        }
        .input-group {
            margin-bottom: 15px;
        }
        .input-group label {
            display: block;
            margin-bottom: 8px;
            color: #555;
            font-weight: bold;
            font-size: 14px;
        }
        .input-group input {
            width: 100%;
            padding: 12px;
            border: 1px solid #dcdfe6;
            border-radius: 6px;
            font-size: 16px;
            box-sizing: border-box; /* 确保 padding 不会撑破宽度 */
            outline: none;
            transition: border-color 0.2s;
        }
        .input-group input:focus {
            border-color: #409eff;
        }
        button {
            width: 100%;
            padding: 14px;
            background-color: #409eff;
            color: white;
            border: none;
            border-radius: 6px;
            font-size: 16px;
            font-weight: bold;
            cursor: pointer;
            margin-top: 10px;
        }
        button:active {
            background-color: #3a8ee6;
        }
        /* 计算结果展示区 */
        .result-box {
            margin-top: 25px;
            padding: 15px;
            background-color: #f0f9eb;
            border: 1px solid #e1f3d8;
            border-radius: 6px;
            color: #67c23a;
            display: none; /* 初始状态隐藏 */
        }
        .result-box p {
            margin: 8px 0;
            font-size: 16px;
            font-weight: bold;
            color: #333;
        }
        .highlight {
            color: #f56c6c;
        }
    </style>
</head>
<body>

    <div class="container">
        <h2>电商业务测算</h2>
        
        <div class="input-group">
            <label>目标GMV (单位：万)</label>
            <input type="number" id="targetGMV" placeholder="请输入数字，如: 100">
        </div>
        <div class="input-group">
            <label>目标ROI (纯数字)</label>
            <input type="number" id="targetROI" placeholder="请输入数字，如: 2.5">
        </div>
        <div class="input-group">
            <label>预估退货率 (%)</label>
            <input type="number" id="returnRate" placeholder="请输入百分比，如: 30">
        </div>
        <div class="input-group">
            <label>福袋及其他费用合计 (单位：万)</label>
            <input type="number" id="otherFees" placeholder="请输入数字，如: 2">
        </div>
        <div class="input-group">
            <label>达人佣金 (%)</label>
            <input type="number" id="commissionRate" placeholder="请输入百分比，如: 20">
        </div>
        <div class="input-group">
            <label>前置费用 (单位：万)</label>
            <input type="number" id="upfrontFee" placeholder="请输入数字，如: 5">
        </div>

        <button onclick="calculateData()">开始计算</button>

        <div class="result-box" id="resultBox">
            <p>退后GMV是（<span class="highlight" id="outNetGMV"></span>）</p>
            <p>费比是（<span class="highlight" id="outExpenseRatio"></span>）</p>
            <p>达人收入是（<span class="highlight" id="outInfluencerIncome"></span>）</p>
        </div>
    </div>

    <script>
        // 这里是 JavaScript（计算逻辑）
        function calculateData() {
            // 1. 获取用户输入的值，如果为空则默认为0
            let GMV = parseFloat(document.getElementById('targetGMV').value) || 0;
            let ROI = parseFloat(document.getElementById('targetROI').value) || 0;
            // 百分比数值需要除以 100 进行计算
            let returnRate = (parseFloat(document.getElementById('returnRate').value) || 0) / 100;
            let otherFees = parseFloat(document.getElementById('otherFees').value) || 0;
            let commissionRate = (parseFloat(document.getElementById('commissionRate').value) || 0) / 100;
            let upfrontFee = parseFloat(document.getElementById('upfrontFee').value) || 0;

            // 防止 ROI 为 0 导致计算错误（除以0）
            if (ROI === 0 && GMV !== 0) {
                alert("目标ROI不能为0！");
                return;
            }

            // 2. 根据公式进行计算
            // 退后GMV = 目标GMV * (1 - 预估退货率)
            let netGMV = GMV * (1 - returnRate);
            
            // 投流消耗 = 目标GMV / 目标ROI
            let adSpend = (ROI > 0) ? (GMV / ROI) : 0;
            
            // 达人佣金金额 = 退后GMV * 达人佣金
            let commissionAmount = netGMV * commissionRate;
            
            // 实际费比 = （投流消耗+达人佣金金额+福袋及其他费用合计+前置费用）/ 退后GMV
            let expenseRatio = 0;
            if (netGMV > 0) {
                expenseRatio = (adSpend + commissionAmount + otherFees + upfrontFee) / netGMV;
            }
            
            // 达人收入 = 前置费用 + 达人佣金金额
            let influencerIncome = upfrontFee + commissionAmount;

            // 3. 将结果输出到页面，并保留两位小数。为了直观，补充了单位说明。
            document.getElementById('outNetGMV').innerText = netGMV.toFixed(2) + "万";
            document.getElementById('outExpenseRatio').innerText = (expenseRatio * 100).toFixed(2) + "%";
            document.getElementById('outInfluencerIncome').innerText = influencerIncome.toFixed(2) + "万";

            // 显示结果框
            document.getElementById('resultBox').style.display = 'block';
        }
    </script>
</body>
</html>
