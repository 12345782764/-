<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>checkin</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Helvetica Neue', Arial, sans-serif;
        }
        
        body {
            background: white;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }
        
        .container {
            width: 100%;
            max-width: 400px;
            background: white;
            border-radius: 10px;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
            overflow: hidden;
            border: 1px solid #e0e0e0;
        }
        
        .header {
            background: #333;
            color: white;
            padding: 20px;
            text-align: center;
        }
        
        .header h1 {
            font-size: 24px;
            font-weight: 400;
        }
        
        .content {
            padding: 30px;
        }
        
        .input-group {
            margin-bottom: 20px;
        }
        
        .input-group input {
            width: 100%;
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 5px;
            font-size: 16px;
            text-align: center;
            letter-spacing: 1px;
        }
        
        .btn {
            background: #333;
            color: white;
            border: none;
            padding: 15px;
            width: 100%;
            border-radius: 5px;
            font-size: 16px;
            cursor: pointer;
        }
        
        .result {
            margin-top: 20px;
            padding: 15px;
            border-radius: 5px;
            text-align: center;
            display: none;
            font-size: 18px;
            font-weight: 500;
        }
        
        .result.valid {
            background: #f8f9fa;
            color: #495057;
            border: 1px solid #dee2e6;
        }
        
        .result.undefined {
            background: #f8f9fa;
            color: #6c757d;
            border: 1px solid #dee2e6;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>checkin</h1>
        </div>
        
        <div class="content">
            <div class="input-group">
                <input type="text" id="product-code" placeholder="输入产品编码" maxlength="17">
            </div>
            
            <button class="btn" id="verify-btn">验证</button>
            
            <div class="result" id="result">
                <span id="result-text"></span>
            </div>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            const verifyBtn = document.getElementById('verify-btn');
            const productCodeInput = document.getElementById('product-code');
            const resultDiv = document.getElementById('result');
            const resultText = document.getElementById('result-text');
            
            // 系列定义 - 在这里添加新的系列和变体
            const seriesMap = {
                '20240609': { 
                    name: '吠/劣', 
                    variants: {
                        '01': { name: '吠', quantityLimit: 25 },  // 吠最多25个
                        '02': { name: '劣', quantityLimit: 18 }   // 劣最多18个
                    }
                },
                '20251027': { 
                    name: '呦/睨', 
                    variants: {
                        '01': { name: '呦', quantityLimit: 15 },  // 呦最多15个
                        '02': { name: '睨', quantityLimit: 20 }   // 睨最多20个
                    }
                }
                // 在这里添加新的系列，格式：
                // 'YYYYMMDD': { 
                //     name: '系列名称', 
                //     variants: {
                //         '01': { name: '变体1名称', quantityLimit: 数量 },
                //         '02': { name: '变体2名称', quantityLimit: 数量 }
                //     }
                // }
            };

            // 肤色映射 - 在这里添加新的肤色
            const skinToneMap = {
                '1': 'mia白',
                '2': 'dd半白', 
                '3': 'soom烧',
                '4': 'soom普',
                '5': '灰肌',
                '6': '牡丹白',
                '7': 'mia粉'
                // 在这里添加新的肤色编码，格式：'编码': '肤色名称'
            };

            verifyBtn.addEventListener('click', function() {
                const productCode = productCodeInput.value.trim();
                
                if (!productCode) {
                    showResult('请输入编码', 'undefined');
                    return;
                }

                if (productCode.length !== 17) {
                    showResult('undefined', 'undefined');
                    return;
                }

                const validationResult = validateProductCode(productCode);
                
                if (validationResult.valid) {
                    const dollName = validationResult.variantInfo.name;
                    showResult(` ${dollName}加入了主人的空间`, 'valid');
                } else {
                    showResult('undefined', 'undefined');
                }
            });
            
            function validateProductCode(code) {
                const seriesCode = code.substring(0, 8);
                const validationCode = code.substring(8, 11);
                const skinTone = code.substring(11, 12);
                const variantCode = code.substring(12, 14);
                const serialNumber = code.substring(14, 17);

                // 检查系列编码
                if (!seriesMap[seriesCode]) {
                    return { valid: false };
                }

                // 检查验证码
                if (validationCode !== '700') {
                    return { valid: false };
                }

                // 检查肤色编码
                if (!skinToneMap[skinTone]) {
                    return { valid: false };
                }

                // 检查变体编码
                const series = seriesMap[seriesCode];
                const variantInfo = series.variants[variantCode];
                if (!variantInfo) {
                    return { valid: false };
                }

                // 检查序列号是否在限制范围内
                const serialNum = parseInt(serialNumber, 10);
                const maxQuantity = variantInfo.quantityLimit;
                
                if (isNaN(serialNum) || serialNum < 1 || serialNum > maxQuantity) {
                    return { valid: false };
                }

                return {
                    valid: true,
                    variantInfo: variantInfo
                };
            }

            function showResult(message, type) {
                resultText.textContent = message;
                resultDiv.className = 'result ' + type;
                resultDiv.style.display = 'block';
            }

            // 输入格式限制
            productCodeInput.addEventListener('input', function(e) {
                e.target.value = e.target.value.replace(/[^\d]/g, '').slice(0, 17);
            });
        });
    </script>
</body>
</html>
