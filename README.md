<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="default">
    <title>💼 保險業務專業試算器 - 新舊保單比較分析</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.0.0"></script>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', 'Microsoft JhengHei', 'Arial', sans-serif;
        }
        
        body {
            background: linear-gradient(135deg, #1a2a6c, #2a5298, #4a90e2);
            min-height: 100vh;
            padding: 20px;
            color: #333;
            -webkit-touch-callout: none;
            -webkit-user-select: none;
            -khtml-user-select: none;
            -moz-user-select: none;
            -ms-user-select: none;
            user-select: none;
            overflow-x: hidden;
        }
        
        /* 列印樣式 */
        @media print {
            * {
                -webkit-print-color-adjust: exact !important;
                print-color-adjust: exact !important;
            }
            
            body {
                background: white !important;
                padding: 0 !important;
                color: #000 !important;
            }
            
            .container {
                box-shadow: none !important;
                border-radius: 0 !important;
                max-width: none !important;
                background: white !important;
            }
            
            .header {
                background: #667eea !important;
                color: white !important;
                page-break-inside: avoid;
            }
            
            .tabs {
                display: none !important;
            }
            
            .tab-content {
                display: block !important;
                page-break-before: always;
                page-break-inside: avoid;
                padding: 20px !important;
            }
            
            .tab-content:first-child {
                page-break-before: auto;
            }
            
            .card {
                page-break-inside: avoid;
                margin-bottom: 20px !important;
                box-shadow: 0 2px 5px rgba(0,0,0,0.1) !important;
                border: 1px solid #ddd !important;
            }
            
            .chart-container {
                height: 300px !important;
                page-break-inside: avoid;
            }
            
            .btn {
                display: none !important;
            }
            
            .footer {
                page-break-before: avoid;
                margin-top: 20px !important;
                background: #f5f5f5 !important;
                color: #333 !important;
            }
            
            @page {
                margin: 2cm;
                size: A4;
            }
        }
        
        .container {
            max-width: 1400px;
            margin: 0 auto;
            background: rgba(255, 255, 255, 0.95);
            border-radius: 25px;
            box-shadow: 0 30px 60px rgba(0, 0, 0, 0.3);
            overflow: hidden;
            backdrop-filter: blur(10px);
        }
        
        .header {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 30px 40px;
            text-align: center;
            position: relative;
            overflow: hidden;
        }
        
        .header::before {
            content: '';
            position: absolute;
            top: -50%;
            left: -50%;
            width: 200%;
            height: 200%;
            background: radial-gradient(circle, rgba(255,255,255,0.1) 0%, rgba(255,255,255,0) 70%);
            transform: rotate(30deg);
        }
        
        .header h1 {
            font-size: 2.8em;
            margin-bottom: 15px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
            position: relative;
            background: linear-gradient(to right, #fff, #a1c4fd);
            -webkit-background-clip: text;
            background-clip: text;
            -webkit-text-fill-color: transparent;
            color: transparent;
            animation: pulse 2s infinite;
        }
        
        @keyframes pulse {
            0% { opacity: 1; }
            50% { opacity: 0.8; }
            100% { opacity: 1; }
        }
        
        .header p {
            font-size: 1.3em;
            opacity: 0.95;
            max-width: 900px;
            margin: 0 auto;
            line-height: 1.7;
            position: relative;
        }
        
        .subtitle {
            background: rgba(255, 255, 255, 0.2);
            display: inline-block;
            padding: 5px 15px;
            border-radius: 20px;
            margin-top: 10px;
            font-weight: bold;
        }
        
        .tabs {
            display: flex;
            background: #f0f4f8;
            border-bottom: 4px solid #667eea;
            box-shadow: 0 4px 10px rgba(0,0,0,0.05);
            position: sticky;
            top: 0;
            z-index: 100;
        }
        
        .tab {
            flex: 1;
            padding: 20px 15px;
            text-align: center;
            cursor: pointer;
            font-weight: bold;
            font-size: 1.15em;
            transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
            position: relative;
            color: #2c3e50;
            min-width: 180px;
        }
        
        .tab:hover {
            background: #e3edff;
            transform: translateY(-2px);
        }
        
        .tab.active {
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
            box-shadow: 0 -4px 15px rgba(102, 126, 234, 0.4);
            transform: translateY(-3px);
        }
        
        .tab.active::after {
            content: '';
            position: absolute;
            bottom: -4px;
            left: 50%;
            transform: translateX(-50%);
            width: 80%;
            height: 4px;
            background: #4facfe;
            border-radius: 2px;
        }
        
        .tab i {
            margin-right: 8px;
            font-size: 1.2em;
        }
        
        .tab-content {
            padding: 30px;
            display: none;
            animation: slideIn 0.6s cubic-bezier(0.4, 0, 0.2, 1);
        }
        
        @keyframes slideIn {
            from { 
                opacity: 0; 
                transform: translateY(20px);
            }
            to { 
                opacity: 1; 
                transform: translateY(0);
            }
        }
        
        .tab-content.active {
            display: block;
        }
        
        .card {
            background: white;
            border-radius: 20px;
            padding: 25px;
            margin: 20px 0;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.12);
            border: 2px solid #eef2f7;
            transition: all 0.3s ease;
            position: relative;
            overflow: hidden;
        }
        
        .card:hover {
            transform: translateY(-5px);
            box-shadow: 0 15px 35px rgba(0, 0, 0, 0.18);
            border-color: #667eea;
        }
        
        .card::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            width: 6px;
            height: 100%;
            background: linear-gradient(to bottom, #667eea, #764ba2);
        }
        
        .card-title {
            font-size: 1.5em;
            color: #2c3e50;
            margin-bottom: 20px;
            padding-bottom: 12px;
            border-bottom: 2px solid #f0f4f8;
            display: flex;
            align-items: center;
        }
        
        .card-title i {
            margin-right: 12px;
            color: #667eea;
            font-size: 1.4em;
        }
        
        .form-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 25px;
            margin: 20px 0;
        }
        
        .form-group {
            margin-bottom: 20px;
        }
        
        .form-label {
            display: block;
            margin-bottom: 8px;
            font-weight: bold;
            color: #2c3e50;
            font-size: 1.05em;
        }
        
        .form-description {
            font-size: 0.9em;
            color: #666;
            margin-top: 4px;
            line-height: 1.4;
        }
        
        input, select, textarea {
            width: 100%;
            padding: 14px;
            border: 2px solid #d1e0f0;
            border-radius: 15px;
            font-size: 1.1em;
            transition: all 0.3s ease;
            background: #f8fafc;
        }
        
        input:focus, select:focus, textarea:focus {
            border-color: #667eea;
            box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.25);
            outline: none;
            background: white;
        }
        
        .currency-switch {
            display: flex;
            gap: 15px;
            margin: 15px 0;
        }
        
        .currency-btn {
            flex: 1;
            padding: 12px;
            border: 2px solid #d1e0f0;
            border-radius: 12px;
            background: #f8fafc;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.3s ease;
            text-align: center;
            font-size: 1.1em;
        }
        
        .currency-btn.active {
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
            border-color: #667eea;
            transform: scale(1.05);
            box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
        }
        
        .btn {
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
            border: none;
            padding: 16px 40px;
            font-size: 1.25em;
            border-radius: 50px;
            cursor: pointer;
            margin: 25px 0;
            transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
            font-weight: bold;
            box-shadow: 0 6px 20px rgba(102, 126, 234, 0.5);
            display: inline-flex;
            align-items: center;
            justify-content: center;
            gap: 10px;
        }
        
        .btn:hover {
            transform: translateY(-3px) scale(1.05);
            box-shadow: 0 10px 25px rgba(102, 126, 234, 0.7);
        }
        
        .btn:active {
            transform: translateY(0) scale(0.98);
        }
        
        .btn i {
            font-size: 1.3em;
        }
        
        .btn-secondary {
            background: linear-gradient(135deg, #4facfe, #00f2fe);
            box-shadow: 0 6px 20px rgba(79, 172, 254, 0.5);
        }
        
        .btn-danger {
            background: linear-gradient(135deg, #ff6b6b, #ee5a24);
            box-shadow: 0 6px 20px rgba(255, 107, 107, 0.5);
        }
        
        .chart-container {
            height: 400px;
            margin: 30px 0;
            background: #f8fafc;
            border-radius: 20px;
            padding: 20px;
            border: 2px solid #eef2f7;
            transition: all 0.3s ease;
        }
        
        .chart-container:hover {
            border-color: #667eea;
            box-shadow: 0 10px 25px rgba(102, 126, 234, 0.2);
        }
        
        .result-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin: 25px 0;
        }
        
        .metric-card {
            background: linear-gradient(135deg, #f8fafc, #eef2f7);
            border-radius: 18px;
            padding: 25px;
            text-align: center;
            box-shadow: 0 8px 20px rgba(0, 0, 0, 0.08);
            transition: all 0.3s ease;
            border: 2px solid #f0f4f8;
        }
        
        .metric-card:hover {
            transform: translateY(-3px);
            border-color: #667eea;
            box-shadow: 0 12px 25px rgba(0, 0, 0, 0.12);
        }
        
        .metric-title {
            font-size: 1.1em;
            color: #666;
            margin-bottom: 10px;
            font-weight: 500;
        }
        
        .metric-value {
            font-size: 2.2em;
            font-weight: bold;
            color: #2c3e50;
            margin: 5px 0;
        }
        
        .metric-positive {
            color: #28a745;
        }
        
        .metric-negative {
            color: #dc3545;
        }
        
        .metric-description {
            font-size: 0.95em;
            color: #666;
            margin-top: 8px;
            line-height: 1.4;
        }
        
        .comparison-table {
            width: 100%;
            border-collapse: collapse;
            margin: 25px 0;
            font-size: 1.05em;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.05);
            border-radius: 15px;
            overflow: hidden;
        }
        
        .comparison-table th, .comparison-table td {
            padding: 16px;
            text-align: center;
            border: 1px solid #eef2f7;
        }
        
        .comparison-table th {
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
            font-weight: bold;
            position: sticky;
            top: 0;
        }
        
        .comparison-table tr:nth-child(even) {
            background: #f8fafc;
        }
        
        .comparison-table tr:hover {
            background: #e3edff;
        }
        
        .highlight {
            background: linear-gradient(135deg, #ffecd2 0%, #fcb69f 100%);
            font-weight: bold;
        }
        
        .positive {
            color: #28a745;
            font-weight: bold;
        }
        
        .negative {
            color: #dc3545;
            font-weight: bold;
        }
        
        .analysis-section {
            background: #e3f2fd;
            padding: 25px;
            border-radius: 20px;
            margin: 25px 0;
            border-left: 6px solid #2196f3;
            position: relative;
            overflow: hidden;
        }
        
        .analysis-section::before {
            content: '';
            position: absolute;
            top: 0;
            right: 0;
            width: 100px;
            height: 100px;
            background: rgba(33, 150, 243, 0.1);
            border-radius: 50%;
        }
        
        .analysis-title {
            font-size: 1.4em;
            color: #1565c0;
            margin-bottom: 15px;
            font-weight: bold;
            position: relative;
        }
        
        .risk-warning {
            background: #fff8e1;
            border: 2px solid #ffc107;
            padding: 25px;
            border-radius: 20px;
            margin: 25px 0;
            position: relative;
        }
        
        .warning-title {
            color: #ff9800;
            font-weight: bold;
            margin-bottom: 15px;
            font-size: 1.3em;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        
        .footer {
            text-align: center;
            padding: 30px;
            background: #2c3e50;
            color: white;
            margin-top: 30px;
            border-top: 4px solid #667eea;
        }
        
        .company-info {
            background: rgba(255, 255, 255, 0.1);
            padding: 20px;
            border-radius: 15px;
            margin: 20px 0;
            text-align: center;
            font-size: 1.1em;
        }
        
        .company-info strong {
            color: #ffd700;
        }
        
        /* iPad 專用優化 */
        @media screen and (min-width: 768px) and (max-width: 1024px) {
            .container {
                max-width: 95%;
                margin: 10px auto;
            }
            
            .header {
                padding: 20px 30px;
            }
            
            .header h1 {
                font-size: 2.5em;
            }
            
            .tab {
                padding: 15px 10px;
                font-size: 1.1em;
                min-width: 150px;
            }
            
            .tab-content {
                padding: 25px;
            }
            
            .form-grid {
                grid-template-columns: repeat(2, 1fr);
                gap: 20px;
            }
            
            .result-grid {
                grid-template-columns: repeat(2, 1fr);
                gap: 15px;
            }
            
            .chart-container {
                height: 350px;
            }
            
            .btn {
                padding: 14px 35px;
                font-size: 1.15em;
                margin: 20px 0;
            }
            
            input, select {
                padding: 12px;
                font-size: 1.05em;
                min-height: 48px;
            }
            
            .currency-btn {
                padding: 12px;
                font-size: 1.05em;
                min-height: 48px;
            }
            
            .comparison-table {
                font-size: 0.95em;
            }
            
            .comparison-table th,
            .comparison-table td {
                padding: 12px 8px;
            }
        }
        
        @media (max-width: 767px) {
            .form-grid, .result-grid {
                grid-template-columns: 1fr;
            }
            .tabs {
                flex-direction: column;
            }
            .header h1 {
                font-size: 2.2em;
            }
            .btn {
                width: 100%;
                margin: 15px 0;
                min-height: 50px;
            }
            .tab {
                min-height: 50px;
            }
            input, select {
                min-height: 48px;
            }
        }
        
        /* 橫向模式優化 */
        @media screen and (orientation: landscape) and (max-width: 1024px) {
            .header {
                padding: 15px 25px;
            }
            
            .header h1 {
                font-size: 2.3em;
                margin-bottom: 10px;
            }
            
            .tab-content {
                padding: 20px;
            }
            
            .chart-container {
                height: 280px;
            }
        }
        
        .animation-bounce {
            animation: bounce 0.5s;
        }
        
        @keyframes bounce {
            0%, 20%, 50%, 80%, 100% {transform: translateY(0);}
            40% {transform: translateY(-15px);}
            60% {transform: translateY(-10px);}
        }
        
        .loading {
            display: inline-block;
            width: 20px;
            height: 20px;
            border: 3px solid rgba(255,255,255,0.3);
            border-radius: 50%;
            border-top-color: white;
            animation: spin 1s linear infinite;
            margin-right: 10px;
        }
        
        @keyframes spin {
            to { transform: rotate(360deg); }
        }
        
        .badge {
            display: inline-block;
            padding: 4px 10px;
            border-radius: 15px;
            font-size: 0.9em;
            font-weight: bold;
            margin-left: 10px;
        }
        
        .badge-positive {
            background: #d4edda;
            color: #155724;
        }
        
        .badge-warning {
            background: #fff3cd;
            color: #856404;
        }
        
        .health-benefits {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 15px;
            margin: 20px 0;
        }
        
        .benefit-card {
            background: #f8fafc;
            border: 2px solid #d1e0f0;
            border-radius: 15px;
            padding: 20px;
            text-align: center;
            transition: all 0.3s ease;
        }
        
        .benefit-card:hover {
            border-color: #667eea;
            transform: scale(1.03);
            box-shadow: 0 8px 20px rgba(102, 126, 234, 0.2);
        }
        
        .benefit-icon {
            font-size: 2.5em;
            color: #667eea;
            margin-bottom: 10px;
        }
        
        .benefit-title {
            font-weight: bold;
            margin-bottom: 8px;
            color: #2c3e50;
        }
        
        /* 觸控優化 */
        .tab, .btn, .currency-btn, .benefit-card {
            -webkit-tap-highlight-color: transparent;
            touch-action: manipulation;
        }
        
        .tab:active, .btn:active, .currency-btn:active {
            transform: scale(0.95);
        }
        
        /* 滾動條優化 */
        ::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }
        
        ::-webkit-scrollbar-track {
            background: #f1f1f1;
            border-radius: 10px;
        }
        
        ::-webkit-scrollbar-thumb {
            background: #667eea;
            border-radius: 10px;
        }
        
        ::-webkit-scrollbar-thumb:hover {
            background: #5a6fd8;
        }
        
        /* iPad特定樣式 */
        @supports (-webkit-touch-callout: none) {
            .container {
                -webkit-overflow-scrolling: touch;
            }
            
            input, select, textarea {
                -webkit-appearance: none;
                appearance: none;
                border-radius: 15px;
            }
            
            .btn {
                -webkit-appearance: none;
                appearance: none;
                cursor: pointer;
            }
        }
        
        /* 橫向滾動表格 */
        .table-wrapper {
            overflow-x: auto;
            -webkit-overflow-scrolling: touch;
            border-radius: 15px;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.05);
        }
        
        /* 全屏模式優化 */
        @media screen and (min-width: 1024px) {
            .container {
                max-width: 1200px;
            }
        }
        
        /* iPad特定優化類 */
        .ipad-optimized {
            -webkit-overflow-scrolling: touch;
        }
        
        .ipad-optimized .container {
            transition: all 0.3s ease;
        }
        
        .ipad-optimized .tab {
            min-height: 48px;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        
        .ipad-optimized input,
        .ipad-optimized select,
        .ipad-optimized textarea {
            min-height: 48px;
            font-size: 16px; /* 防止縮放 */
        }
        
        .ipad-optimized .btn {
            min-height: 48px;
            font-size: 16px;
        }
        
        .ipad-optimized .currency-btn {
            min-height: 48px;
            font-size: 16px;
        }
        
        /* 防止縮放的輸入框 */
        input[type="number"],
        input[type="text"],
        select {
            font-size: 16px !important;
        }
        
        /* 表格橫向滾動指示 */
        .table-wrapper {
            position: relative;
        }
        
        .table-wrapper::after {
            content: '← 左右滑動查看更多 →';
            position: absolute;
            bottom: -25px;
            left: 50%;
            transform: translateX(-50%);
            font-size: 0.9em;
            color: #666;
            opacity: 0.7;
        }
        
        @media (min-width: 1024px) {
            .table-wrapper::after {
                display: none;
            }
        }
        
        /* 錯誤提示樣式 */
        .error-message {
            color: #dc3545;
            font-size: 0.9em;
            margin-top: 5px;
            display: none;
            animation: shake 0.3s;
        }
        
        .error-message.show {
            display: block;
        }
        
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            25% { transform: translateX(-10px); }
            75% { transform: translateX(10px); }
        }
        
        .input-error {
            border-color: #dc3545 !important;
            background-color: #fff5f5 !important;
        }
        
        .input-valid {
            border-color: #28a745 !important;
        }
        
        /* 工具提示 */
        .tooltip {
            position: relative;
            display: inline-block;
            cursor: help;
            margin-left: 5px;
        }
        
        .tooltip .tooltiptext {
            visibility: hidden;
            width: 250px;
            background-color: #2c3e50;
            color: #fff;
            text-align: left;
            border-radius: 10px;
            padding: 12px;
            position: absolute;
            z-index: 1000;
            bottom: 125%;
            left: 50%;
            margin-left: -125px;
            opacity: 0;
            transition: opacity 0.3s;
            font-size: 0.9em;
            line-height: 1.4;
            box-shadow: 0 4px 15px rgba(0,0,0,0.3);
        }
        
        .tooltip .tooltiptext::after {
            content: "";
            position: absolute;
            top: 100%;
            left: 50%;
            margin-left: -5px;
            border-width: 5px;
            border-style: solid;
            border-color: #2c3e50 transparent transparent transparent;
        }
        
        .tooltip:hover .tooltiptext {
            visibility: visible;
            opacity: 1;
        }
        
        /* 情境分析樣式 */
        .scenario-tabs {
            display: flex;
            gap: 10px;
            margin: 20px 0;
            flex-wrap: wrap;
        }
        
        .scenario-btn {
            flex: 1;
            min-width: 150px;
            padding: 12px 20px;
            border: 2px solid #d1e0f0;
            border-radius: 12px;
            background: white;
            cursor: pointer;
            transition: all 0.3s ease;
            font-weight: bold;
            text-align: center;
        }
        
        .scenario-btn.active {
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
            border-color: #667eea;
            transform: translateY(-2px);
            box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
        }
        
        .scenario-btn:hover:not(.active) {
            background: #f0f4f8;
            transform: translateY(-2px);
        }
        
        /* 智能建議卡片 */
        .ai-suggestion {
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
            padding: 25px;
            border-radius: 20px;
            margin: 25px 0;
            box-shadow: 0 10px 30px rgba(102, 126, 234, 0.3);
            position: relative;
            overflow: hidden;
        }
        
        .ai-suggestion::before {
            content: '🤖';
            position: absolute;
            top: -20px;
            right: -20px;
            font-size: 120px;
            opacity: 0.1;
        }
        
        .ai-suggestion-title {
            font-size: 1.5em;
            font-weight: bold;
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        
        .ai-suggestion-content {
            font-size: 1.1em;
            line-height: 1.8;
            position: relative;
        }
        
        .ai-suggestion-content strong {
            color: #ffd700;
        }
        
        /* 雷達圖容器 */
        .radar-container {
            height: 400px;
            margin: 30px 0;
            background: white;
            border-radius: 20px;
            padding: 20px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
        }
        
        /* 損益平衡分析 */
        .breakeven-card {
            background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
            color: white;
            padding: 25px;
            border-radius: 20px;
            margin: 20px 0;
            text-align: center;
        }
        
        .breakeven-year {
            font-size: 3em;
            font-weight: bold;
            margin: 15px 0;
        }
        
        .breakeven-description {
            font-size: 1.1em;
            opacity: 0.95;
        }
        
        /* 完整預覽模式 */
        .show-all-content .tab-content {
            border-bottom: 3px solid #667eea;
            margin-bottom: 30px;
            padding-bottom: 30px;
        }
        
        .show-all-content .tab-content:last-child {
            border-bottom: none;
            margin-bottom: 0;
        }
        
        .show-all-content .tab-content::before {
            content: attr(data-title);
            display: block;
            font-size: 1.8em;
            font-weight: bold;
            color: #667eea;
            margin-bottom: 20px;
            padding: 15px;
            background: linear-gradient(135deg, #f8fafc, #eef2f7);
            border-radius: 15px;
            border-left: 6px solid #667eea;
        }
        
        /* 固定按鈕位置 */
        .fixed-actions {
            position: fixed;
            bottom: 20px;
            right: 20px;
            z-index: 1000;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        
        .fixed-actions .btn {
            margin: 0;
            padding: 12px 20px;
            font-size: 0.9em;
            border-radius: 25px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.2);
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>💼 保險業務專業試算器</h1>
            <p>精準分析新舊保單效益 · 即時計算報酬率 · 可視化數據比較</p>
            <div class="subtitle">📊 基於國泰人壽2024年獲利1,111.9億元數據設計 · 客觀分析工具</div>
        </div>
        
        <div class="tabs">
            <div class="tab active" onclick="switchTab(1)">
                <i>🎯</i>參數設定
            </div>
            <div class="tab" onclick="switchTab(2)">
                <i>💰</i>現金價值
            </div>
            <div class="tab" onclick="switchTab(3)">
                <i>📈</i>投資效益
            </div>
            <div class="tab" onclick="switchTab(4)">
                <i>🎯</i>專業建議
            </div>
        </div>
        
        <!-- 第1頁：參數設定 -->
        <div class="tab-content active" id="tab1" data-title="🎯 第1頁：參數設定">
            <div class="card">
                <div class="card-title">
                    <i>📊</i>舊保單參數設定
                </div>
                
                <div class="form-group">
                    <label class="form-label">保單幣別</label>
                    <div class="currency-switch">
                        <div class="currency-btn active" onclick="switchCurrency('old', 'NTD')">台幣儲蓄險</div>
                        <div class="currency-btn" onclick="switchCurrency('old', 'USD')">美元儲蓄險</div>
                    </div>
                    <div class="form-description">選擇您現有舊保單的幣別（已滿期且超過10年）</div>
                </div>
                
                <div class="form-grid">
                    <div class="form-group">
                        <label class="form-label">
                            客戶年齡
                            <span class="tooltip">ℹ️
                                <span class="tooltiptext">年齡會影響保障倍數計算。年齡越小，保障倍數越高（最高3倍）。建議範圍：20-70歲。</span>
                            </span>
                        </label>
                        <input type="number" id="clientAge" value="45" min="20" max="70" placeholder="請輸入客戶年齡" onblur="validateInput(this, 20, 70, 'age')">
                        <div class="error-message" id="error-clientAge"></div>
                        <div class="form-description">年齡影響保障倍數和配息計算</div>
                    </div>
                    
                    <div class="form-group">
                        <label class="form-label">
                            舊保單目前現金價值（萬元）
                            <span class="tooltip">ℹ️
                                <span class="tooltiptext">請輸入保單目前的解約金或現金價值。這是評估是否轉換的重要基準。建議範圍：10-5000萬元。</span>
                            </span>
                        </label>
                        <input type="number" id="oldCashValueInput" value="150" step="1" min="10" placeholder="現金價值（萬元）" onblur="validateInput(this, 10, 5000, 'value')">
                        <div class="error-message" id="error-oldCashValueInput"></div>
                        <div class="form-description">舊保單目前可解約的現金價值</div>
                    </div>
                       
                    <div class="form-group">
                        <label class="form-label">舊保單宣告利率 (%)</label>
                        <input type="number" id="oldRate" value="2.5" step="0.1" min="0" max="6" placeholder="宣告利率">
                        <div class="form-description">舊保單目前配息利率（保證+非保證）</div>
                    </div>

                    <div class="form-group">
                        <label class="form-label">舊保單預定利率 (%) <span class="badge badge-warning" style="font-size: 0.8em;">保證</span></label>
                        <input type="number" id="oldGuaranteedRate" value="1.5" step="0.1" min="0" max="6" placeholder="預定利率">
                        <div class="form-description">保單條款承諾的最低保證利率</div>
                    </div>
                    
                    <div class="form-group">
                        <label class="form-label">舊保單年配息金額（萬元）</label>
                        <input type="number" id="oldAnnualPayout" value="3.75" step="0.1" min="0" placeholder="年配息金額（萬元）">
                        <div class="form-description">舊保單每年實際配息金額（現金價值 × 利率）</div>
                    </div>
                    
                    <div class="form-group">
                        <label class="form-label">舊保單保障額度（萬元）</label>
                        <input type="number" id="oldCoverage" value="200" step="10" min="50" placeholder="保障額度（萬元）">
                        <div class="form-description">舊保單的壽險保障金額</div>
                    </div>
                </div>
            </div>
            
            <div class="card">
                <div class="card-title">
                    <i>✨</i>新保單參數設定
                </div>
                
                <div class="form-group">
                    <label class="form-label">新保單幣別</label>
                    <div class="currency-switch">
                        <div class="currency-btn" onclick="switchCurrency('new', 'NTD')">台幣儲蓄險</div>
                        <div class="currency-btn active" onclick="switchCurrency('new', 'USD')">美元儲蓄險</div>
                    </div>
                    <div class="form-description">推薦的新商品幣別（目前美元利率較高）</div>
                </div>

                <div class="form-group" id="exchangeRateGroup">
                    <label class="form-label">
                        匯率設定 (USD/NTD)
                        <span class="tooltip">ℹ️
                            <span class="tooltiptext">若涉及美元保單，請輸入預估匯率以進行換算比較。</span>
                        </span>
                    </label>
                    <input type="number" id="exchangeRate" value="32.5" step="0.1" min="20" max="40" placeholder="匯率">
                    <div class="form-description">用於將美元數值換算為台幣進行比較</div>
                </div>
                
                <div class="form-grid">
                    <div class="form-group">
                        <label class="form-label">
                            新保單年繳保費（萬元）
                            <span class="tooltip">ℹ️
                                <span class="tooltiptext">建議用舊保單現金價值的20-30%購買新保單，既能享受新保單優勢，又保留舊保單穩定性。範圍：5-500萬元/年。</span>
                            </span>
                        </label>
                        <input type="number" id="newPremium" value="30" step="1" min="5" placeholder="年繳保費（萬元）" onblur="validateInput(this, 5, 500, 'premium')">
                        <div class="error-message" id="error-newPremium"></div>
                        <div class="form-description">建議用舊保單現金價值的20%購買新保單</div>
                    </div>
                    
                    <div class="form-group">
                        <label class="form-label">
                            新保單宣告利率 (%)
                            <span class="tooltip">ℹ️
                                <span class="tooltiptext">宣告利率不等於保證利率！保險公司每月可調整。當前美元保單約4.0-4.5%，台幣約2.0-2.5%。</span>
                            </span>
                        </label>
                        <input type="number" id="newRate" value="4.2" step="0.1" min="0" max="6" placeholder="宣告利率" onblur="validateInput(this, 0, 6, 'rate')">
                        <div class="error-message" id="error-newRate"></div>
                        <div class="form-description">新保單目前宣告利率（2024-2025年水平）</div>
                    </div>

                    <div class="form-group">
                        <label class="form-label">新保單預定利率 (%) <span class="badge badge-warning" style="font-size: 0.8em;">保證</span></label>
                        <input type="number" id="newGuaranteedRate" value="2.5" step="0.1" min="0" max="6" placeholder="預定利率">
                        <div class="form-description">新保單的最低保證利率（保守評估基準）</div>
                    </div>
                    
                    <div class="form-group">
                        <label class="form-label">保障倍數</label>
                        <input type="number" id="newCoverageMultiple" value="2.0" step="0.1" min="1.0" max="3.0" placeholder="保障倍數">
                        <div class="form-description">新保單保障額度為保費的倍數（年齡越小倍數越高）</div>
                    </div>
                    
                    <div class="form-group">
                        <label class="form-label">保單年期</label>
                        <select id="policyTerm">
                            <option value="6">6年期</option>
                            <option value="10">10年期</option>
                            <option value="15">15年期</option>
                            <option value="20">20年期</option>
                        </select>
                        <div class="form-description">新保單的繳費年期</div>
                    </div>
                </div>
                
                <div class="form-group">
                    <label class="form-label">配息方式</label>
                    <div class="currency-switch">
                        <div class="currency-btn active" onclick="switchPayout('none')">不領回（複利滾存）</div>
                        <div class="currency-btn" onclick="switchPayout('annual')">年配息（當年金領）</div>
                    </div>
                    <div class="form-description">選擇配息領取方式，影響現金流和總報酬</div>
                </div>
            </div>
            
            <div class="card">
                <div class="card-title">
                    <i>🏥</i>附加醫療保障
                </div>
                
                <div class="health-benefits">
                    <div class="benefit-card">
                        <div class="benefit-icon">💔</div>
                        <div class="benefit-title">癌症保險金（萬元）</div>
                        <input type="number" id="cancerBenefit" value="100" step="10" min="0" placeholder="癌症給付（萬元）">
                    </div>
                    
                    <div class="benefit-card">
                        <div class="benefit-icon">⚡</div>
                        <div class="benefit-title">重大疾病（萬元）</div>
                        <input type="number" id="criticalIllness" value="60" step="10" min="0" placeholder="重疾給付（萬元）">
                    </div>
                    
                    <div class="benefit-card">
                        <div class="benefit-icon">🏠</div>
                        <div class="benefit-title">長期照護（萬元）</div>
                        <input type="number" id="longTermCare" value="3" step="0.5" min="0" placeholder="月領（萬元）">
                    </div>
                </div>
                
                <div class="form-description">
                    新美元保單附加保障功能，舊保單通常不具備這些功能，大幅提升CP值
                </div>
            </div>
            
            <button class="btn" onclick="calculateAll(); return false;">
                <i>⚡</i>立即計算分析
            </button>
            
            <div class="company-info">
                💡 <strong>數據說明：</strong>本分析假設舊保單已滿期且無解約費用。新保單價值為預估值，實際數值請以建議書為準。
            </div>
        </div>
        
        <!-- 第2頁：現金價值比較 -->
        <div class="tab-content" id="tab2" data-title="💰 第2頁：現金價值比較">
            <div class="card">
                <div class="card-title">
                    <i>💰</i>現金價值與IRR分析
                </div>
                
                <div class="result-grid">
                    <div class="metric-card">
                        <div class="metric-title">舊保單現金價值</div>
                        <div class="metric-value metric-positive" id="oldCashValue">150.0萬元</div>
                        <div class="metric-description">已累積多年，保證價值</div>
                    </div>
                    
                    <div class="metric-card">
                        <div class="metric-title">舊保單年配息</div>
                        <div class="metric-value metric-positive" id="oldPayoutDisplay">3.75萬元</div>
                        <div class="metric-description">每年實際配息金額</div>
                    </div>
                    
                    <div class="metric-card">
                        <div class="metric-title">新保單現金價值</div>
                        <div class="metric-value" id="newCashValue">32.5萬元</div>
                        <div class="metric-description">6年期滿後價值（含配息）</div>
                    </div>
                    
                    <div class="metric-card">
                        <div class="metric-title">舊保單IRR</div>
                        <div class="metric-value metric-positive" id="oldIRR">2.50%</div>
                        <div class="metric-description">內部報酬率（保證+非保證）</div>
                    </div>
                    
                    <div class="metric-card">
                        <div class="metric-title">新保單IRR</div>
                        <div class="metric-value metric-positive" id="newIRR">4.20%</div>
                        <div class="metric-description">內部報酬率（基於當前利率）</div>
                    </div>
                </div>
                
                <div class="chart-container">
                    <canvas id="cashValueChart"></canvas>
                </div>
                
                <div class="risk-warning">
                    <div class="warning-title">⚠️ 重要提醒</div>
                    <p>• 舊保單現金價值已累積多年，新保單需要重新累積，<strong>短期內舊保單絕對優勢</strong></p>
                    <p>• 新保單4.2%宣告利率<strong>不保證</strong>，可能隨市場利率下降，舊保單2.5%相對穩定</p>
                    <p>• 美元保單需承擔<mark>匯率風險</mark>，若台幣升值，實際台幣報酬會降低</p>
                </div>
            </div>
        </div>
        
        <!-- 第3頁：投資效益分析 -->
        <div class="tab-content" id="tab3" data-title="📈 第3頁：投資效益分析">
            <div class="card">
                <div class="card-title">
                    <i>📈</i>投資效益與配息分析
                </div>
                
                <div class="result-grid">
                    <div class="metric-card">
                        <div class="metric-title">年配息金額</div>
                        <div class="metric-value" id="annualPayout">13,650美元</div>
                        <div class="metric-description">新保單年配息（基於4.2%利率）</div>
                    </div>
                    
                    <div class="metric-card">
                        <div class="metric-title">回收年限</div>
                        <div class="metric-value" id="paybackPeriod">8.2年</div>
                        <div class="metric-description">從新保單開始計算的回收期</div>
                    </div>
                    
                    <div class="metric-card">
                        <div class="metric-title">10年總報酬</div>
                        <div class="metric-value" id="totalReturn10yr">47.8萬</div>
                        <div class="metric-description">新保單10年後累計價值</div>
                    </div>
                    
                    <div class="metric-card">
                        <div class="metric-title">20年總報酬</div>
                        <div class="metric-value" id="totalReturn20yr">71.2萬</div>
                        <div class="metric-description">新保單20年後累計價值</div>
                    </div>
                </div>
                
                <div class="chart-container">
                    <canvas id="returnChart"></canvas>
                </div>
                
                <div class="analysis-section">
                    <div class="analysis-title">🎯 配息方式比較分析</div>
                    <p><strong>不領回（複利滾存）：</strong>適合長期儲蓄，複利效果最大，20年後價值最高</p>
                    <p><strong>年配息（當年金領）：</strong>適合退休規劃，每年有穩定現金流，但總報酬較低</p>
                    <p><strong>最佳策略：</strong>45歲以下客戶建議複利滾存，55歲以上客戶建議年配息當退休金補充</p>
                </div>
                
                <div class="card">
                    <div class="card-title">
                        <i>🎲</i>情境模擬分析
                    </div>
                    
                    <div class="scenario-tabs">
                        <div class="scenario-btn active" onclick="switchScenario('current')">
                            📊 當前情境<br><small>利率維持</small>
                        </div>
                        <div class="scenario-btn" onclick="switchScenario('optimistic')">
                            📈 樂觀情境<br><small>利率上升0.5%</small>
                        </div>
                        <div class="scenario-btn" onclick="switchScenario('conservative')">
                            📉 保守情境<br><small>利率下降0.5%</small>
                        </div>
                        <div class="scenario-btn" onclick="switchScenario('worst')">
                            ⚠️ 最差情境<br><small>利率下降1.5%</small>
                        </div>
                    </div>
                    
                    <div class="result-grid" id="scenarioResults">
                        <div class="metric-card">
                            <div class="metric-title">10年後總價值</div>
                            <div class="metric-value" id="scenario10yr">47.8萬元</div>
                        </div>
                        <div class="metric-card">
                            <div class="metric-title">20年後總價值</div>
                            <div class="metric-value" id="scenario20yr">71.2萬元</div>
                        </div>
                        <div class="metric-card">
                            <div class="metric-title">預期IRR</div>
                            <div class="metric-value" id="scenarioIRR">4.20%</div>
                        </div>
                        <div class="metric-card">
                            <div class="metric-title">風險評級</div>
                            <div class="metric-value" id="scenarioRisk">中等</div>
                        </div>
                    </div>
                    
                    <div class="chart-container">
                        <canvas id="scenarioChart"></canvas>
                    </div>
                </div>
                
                <div class="breakeven-card">
                    <div class="breakeven-description">💡 損益平衡點分析</div>
                    <div class="breakeven-year" id="breakevenYear">12.5年</div>
                    <div class="breakeven-description">新保單預計在此時間點後開始超越舊保單總價值</div>
                </div>
            </div>
        </div>
        
        <!-- 第4頁：專業建議 -->
        <div class="tab-content" id="tab4" data-title="🎯 第4頁：專業建議">
            <div class="ai-suggestion">
                <div class="ai-suggestion-title">
                    🤖 AI智能分析建議
                </div>
                <div class="ai-suggestion-content" id="aiSuggestion">
                    正在分析您的保單數據，生成專業建議...
                </div>
            </div>
            
            <div class="card">
                <div class="card-title">
                    <i>📊</i>多維度雷達圖比較
                </div>
                <div class="radar-container">
                    <canvas id="radarChart"></canvas>
                </div>
                <div class="form-description" style="text-align: center; margin-top: 15px;">
                    雷達圖綜合比較：收益率、保障額度、流動性、穩定性、附加價值等五大維度
                </div>
            </div>
            
            <div class="card">
                <div class="card-title">
                    <i>🎯</i>專業建議與風險評估
                </div>
                
                <div class="analysis-section">
                    <div class="analysis-title">✅ 建議策略：分批轉換，不要全解約</div>
                    <p>1. <strong>保留舊保單</strong>：現金價值150萬已累積多年，保證利率2.5%仍是優質資產</p>
                    <p>2. <strong>加買新保單</strong>：用新增資金5萬/年購買美元保單，補足保障缺口和退休規劃</p>
                    <p>3. <strong>資金分配</strong>：建議70%資金保留舊保單，30%資金加買新保單，風險分散</p>
                </div>
                
                <div class="analysis-section">
                    <div class="analysis-title">💰 財務效益分析</div>
                    <p>• <strong>5年內</strong>：舊保單現金價值絕對優勢，不建議解約，差異達117.5萬</p>
                    <p>• <strong>10年後</strong>：新保單復利效果開始顯現，但舊保單仍有領先（192萬 vs 47.8萬）</p>
                    <p>• <strong>20年後</strong>：若利率維持4.2%，新保單可能追上，但需承擔匯率和利率風險</p>
                    <p>• <strong>整體CP值</strong>：新保單附加功能（癌症50萬+重疾30萬+長照）CP值極高</p>
                </div>
                
                <div class="risk-warning">
                    <div class="warning-title">🚨 業務員應避免的話術</div>
                    <p>❌ <strong>「新保單一定比舊保單划算」</strong> - 需客觀分析客戶年齡、風險承受度、保障需求</p>
                    <p>❌ <strong>「保險公司不會虧錢」</strong> - 應解釋獲利模式：國泰人壽2024年稅後淨利671.8億元 [[3]]</p>
                    <p>❌ <strong>「現在不換以後沒機會」</strong> - 這是高壓銷售話術，可能違法</p>
                    <p>✅ <strong>正確話術</strong>：「讓我們分析您的需求，看是否需要補充保障或分散投資風險」</p>
                </div>
                
                <div class="table-wrapper">
                <table class="comparison-table">
                    <thead>
                        <tr>
                            <th>評估項目</th>
                            <th>舊保單（台幣）</th>
                            <th>新保單（美元）</th>
                            <th>建議</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td>投資報酬率</td>
                            <td>2.5% (保證)</td>
                            <td>4.2% (非保證)</td>
                            <td class="positive">新保單較高</td>
                        </tr>
                        <tr>
                            <td>流動性</td>
                            <td>✅ 隨時可解約</td>
                            <td>❌ 前5年解約虧損大</td>
                            <td class="negative">舊保單較佳</td>
                        </tr>
                        <tr>
                            <td>保障功能</td>
                            <td>❌ 僅基本壽險</td>
                            <td>✅ 癌症+重疾+長照</td>
                            <td class="positive">新保單CP值高</td>
                        </tr>
                        <tr>
                            <td>匯率風險</td>
                            <td>❌ 無</td>
                            <td>✅ 高風險</td>
                            <td class="negative">舊保單穩健</td>
                        </tr>
                        <tr>
                            <td>退休規劃</td>
                            <td>❌ 無年金功能</td>
                            <td>✅ 可轉年金</td>
                            <td class="positive">新保單適合</td>
                        </tr>
                    </tbody>
                </table>
                </div>
                
                <div style="display: flex; gap: 15px; flex-wrap: wrap; margin-top: 25px;">
                    <button class="btn btn-secondary" onclick="generateReport()">
                        <i>📋</i>生成完整分析報告
                    </button>
                    
                    <button class="btn" onclick="printAllPages()">
                        <i>🖨️</i>列印完整內容
                    </button>
                    
                    <button class="btn btn-danger" onclick="exportToPDF()">
                        <i>📄</i>匯出PDF
                    </button>
                    
                    <button class="btn" onclick="showAllContent()" style="background: linear-gradient(135deg, #28a745, #20c997);">
                        <i>📱</i>iPad完整預覽
                    </button>
                </div>
            </div>
        </div>
        
        <div class="footer">
            <p>© 2025 保險專業分析工具 | 數據來源：國泰金控2024年財報 [[3]] [[5]] [[8]]</p>
            <p style="margin-top: 10px; font-size: 0.95em; opacity: 0.9;">
                本工具基於國泰金控2024年稅後淨利1,111.9億元數據設計，僅供參考，實際保單內容以保險公司條款為準。
            </p>
            <p style="margin-top: 8px; font-size: 0.9em; opacity: 0.8;">
                ⚠️ 投資有風險，保單解約可能產生損失，請謹慎評估個人財務狀況。
            </p>
        </div>
    </div>

    <script>
        // 全局變數
        let oldCurrency = 'NTD';
        let newCurrency = 'USD';
        let payoutMethod = 'none';
        let currentScenario = 'current';
        let scenarioChartInstance = null;
        let radarChartInstance = null;
        
        function switchTab(tabNumber) {
            // 移除所有 active class
            document.querySelectorAll('.tab').forEach(tab => tab.classList.remove('active'));
            document.querySelectorAll('.tab-content').forEach(content => content.classList.remove('active'));
            
            // 添加 active class 到選中的 tab 和內容
            document.querySelector(`.tabs .tab:nth-child(${tabNumber})`).classList.add('active');
            document.getElementById(`tab${tabNumber}`).classList.add('active');
            
            // 如果是結果頁面且還沒計算，自動計算
            if (tabNumber > 1) {
                calculateAll();
            }
        }
        
        function switchCurrency(type, currency) {
            if (type === 'old') {
                oldCurrency = currency;
                document.querySelectorAll('.currency-btn')[0].classList.toggle('active', currency === 'NTD');
                document.querySelectorAll('.currency-btn')[1].classList.toggle('active', currency === 'USD');
            } else {
                newCurrency = currency;
                document.querySelectorAll('.currency-btn')[2].classList.toggle('active', currency === 'NTD');
                document.querySelectorAll('.currency-btn')[3].classList.toggle('active', currency === 'USD');
            }
        }
        
        function switchPayout(method) {
            payoutMethod = method;
            document.querySelectorAll('.currency-btn')[4].classList.toggle('active', method === 'none');
            document.querySelectorAll('.currency-btn')[5].classList.toggle('active', method === 'annual');
            calculateAll();
        }
        
        function calculateIRR(cashFlows) {
            // 簡化版IRR計算（實際應用需更精確演算法）
            const rate = 0.042; // 假設利率
            return (rate * 100).toFixed(2);
        }
        
        function calculateCashValue() {
            const oldCashValueInput = parseFloat(document.getElementById('oldCashValueInput').value) || 150;
            const oldRate = parseFloat(document.getElementById('oldRate').value);
            const oldGuaranteedRate = parseFloat(document.getElementById('oldGuaranteedRate').value) || 1.5;
            
            const newPremium = parseFloat(document.getElementById('newPremium').value);
            const newRate = parseFloat(document.getElementById('newRate').value);
            const newGuaranteedRate = parseFloat(document.getElementById('newGuaranteedRate').value) || 2.5;
            
            const policyTerm = parseInt(document.getElementById('policyTerm').value);
            const clientAge = parseInt(document.getElementById('clientAge').value);
            const exchangeRate = parseFloat(document.getElementById('exchangeRate').value) || 32.5;
            
            // 匯率換算邏輯：統一換算成台幣(NTD)進行比較
            let oldCashValueNTD = oldCashValueInput;
            if (oldCurrency === 'USD') {
                oldCashValueNTD = oldCashValueInput * exchangeRate;
            }

            let newPremiumNTD = newPremium;
            if (newCurrency === 'USD') {
                newPremiumNTD = newPremium * exchangeRate;
            }

            // 自動計算舊保單年配息 (顯示用，保持原幣別或標註)
            const calculatedOldPayout = oldCashValueInput * (oldRate / 100);
            document.getElementById('oldAnnualPayout').value = calculatedOldPayout.toFixed(2);
            
            // 舊保單現金價值（NTD）
            const oldCashValue = oldCashValueNTD;
            
            // 新保單現金價值（NTD）
            const newTotalPremiumNTD = newPremiumNTD * policyTerm;
            
            // 計算新保單價值 (基於預定利率的保守估計)
            // 使用複利公式：本金 * (1 + 預定利率)^年期
            // 但需考慮前幾年的費用率，這裡做一個簡易的費用扣除模型
            // 假設前6年費用較高，之後純複利
            
            let newCashValueNTD;
            let newCashValueDeclaredNTD; // 宣告利率版本(參考用)

            // 簡易模型：滿期時約等於 本金 * (1 + 預定利率 * 年期 * 0.8) -> 粗略估算扣費後
            // 更精確一點：
            // 6年期：第6年末約保本+一點利息 (視預定利率而定)
            
            // 基準因子 (假設預定利率2.0%時的滿期價值係數)
            let baseFactor = 1.0;
            if (policyTerm === 6) baseFactor = 1.05;
            else if (policyTerm === 10) baseFactor = 1.15;
            else if (policyTerm === 15) baseFactor = 1.35;
            else baseFactor = 1.55;

            // 根據實際預定利率調整
            // 差異率
            const rateDiff = (newGuaranteedRate - 2.0) / 100;
            const adjustedFactor = baseFactor + (rateDiff * policyTerm);
            
            newCashValueNTD = newTotalPremiumNTD * adjustedFactor;

            // 計算宣告利率版本 (用於比較)
            const rateDiffDeclared = (newRate - 2.0) / 100;
            const adjustedFactorDeclared = baseFactor + (rateDiffDeclared * policyTerm);
            newCashValueDeclaredNTD = newTotalPremiumNTD * adjustedFactorDeclared;
            
            // 考慮年齡對保障倍數的影響
            const ageFactor = Math.max(1.0, 3.0 - (clientAge - 30) * 0.1);
            const actualCoverageMultiple = Math.min(parseFloat(document.getElementById('newCoverageMultiple').value), ageFactor);
            
            return {
                oldCashValue: oldCashValue, // NTD
                newCashValue: newCashValueNTD, // NTD (Guaranteed)
                newCashValueDeclared: newCashValueDeclaredNTD, // NTD (Declared)
                oldIRR: oldRate.toFixed(2),
                newIRR: newRate.toFixed(2),
                oldGuaranteedIRR: oldGuaranteedRate.toFixed(2),
                newGuaranteedIRR: newGuaranteedRate.toFixed(2),
                ageFactor,
                actualCoverageMultiple,
                newTotalPremiumNTD,
                oldGuaranteedRate,
                newGuaranteedRate
            };
        }
        
        function calculateReturns() {
            const newPremium = parseFloat(document.getElementById('newPremium').value);
            const newRate = parseFloat(document.getElementById('newRate').value);
            const newGuaranteedRate = parseFloat(document.getElementById('newGuaranteedRate').value) || 2.5;
            const policyTerm = parseInt(document.getElementById('policyTerm').value);
            const exchangeRate = parseFloat(document.getElementById('exchangeRate').value) || 32.5;
            
            let newPremiumNTD = newPremium;
            if (newCurrency === 'USD') {
                newPremiumNTD = newPremium * exchangeRate;
            }

            const newTotalPremiumNTD = newPremiumNTD * policyTerm;
            let annualPayoutNTD;
            
            // 使用預定利率計算保證配息 (若有)
            // 這裡假設年配息是基於宣告利率，但我們也提供一個保證配息的參考
            // 為了符合用戶「以預定利率為基準」的要求，這裡主要顯示保證配息
            
            if (payoutMethod === 'annual') {
                // 年配息金額（台幣萬元）- 使用預定利率計算保證配息
                annualPayoutNTD = newTotalPremiumNTD * (newGuaranteedRate / 100);
            } else {
                annualPayoutNTD = 0;
            }
            
            // 10年總報酬（使用預定利率計算保證價值）
            const return10yr = newTotalPremiumNTD * Math.pow(1 + newGuaranteedRate/100, 10);
            const return20yr = newTotalPremiumNTD * Math.pow(1 + newGuaranteedRate/100, 20);
            
            // 回收年限（簡化計算，基於預定利率）
            const paybackPeriod = newTotalPremiumNTD / (newTotalPremiumNTD * newGuaranteedRate/100);
            
            return {
                annualPayout: annualPayoutNTD, // NTD (Guaranteed)
                totalReturn10yr: return10yr,
                totalReturn20yr: return20yr,
                paybackPeriod: paybackPeriod.toFixed(1)
            };
        }
        
        function calculateAll(autoSwitch = true) {
            // 顯示loading效果
            const btn = document.querySelector('.btn');
            const originalText = btn.innerHTML;
            btn.innerHTML = '<span class="loading"></span>計算中...';
            btn.classList.add('animation-bounce');
            
            setTimeout(() => {
                // 計算現金價值
                const cashValueData = calculateCashValue();
                const oldPayout = parseFloat(document.getElementById('oldAnnualPayout').value) || 0;
                
                document.getElementById('oldCashValue').textContent = `${cashValueData.oldCashValue.toFixed(1)}萬元`;
                document.getElementById('oldPayoutDisplay').textContent = `${oldPayout.toFixed(2)}萬元`;
                document.getElementById('newCashValue').textContent = `${cashValueData.newCashValue.toFixed(1)}萬元`;
                document.getElementById('oldIRR').textContent = `${cashValueData.oldIRR}%`;
                document.getElementById('newIRR').textContent = `${cashValueData.newIRR}%`;
                
                // 計算投資效益
                const returnData = calculateReturns();
                document.getElementById('annualPayout').textContent = payoutMethod === 'annual' ? 
                    `${returnData.annualPayout.toFixed(1)}萬元` : 
                    '複利滾存';
                document.getElementById('totalReturn10yr').textContent = `${returnData.totalReturn10yr.toFixed(1)}萬元`;
                document.getElementById('totalReturn20yr').textContent = `${returnData.totalReturn20yr.toFixed(1)}萬元`;
                document.getElementById('paybackPeriod').textContent = `${returnData.paybackPeriod}年`;
                
                // 更新圖表
                updateCharts();
                
                // 更新情境分析
                calculateScenario(currentScenario);
                
                // 更新雷達圖
                updateRadarChart();
                
                // 生成AI建議
                generateAISuggestion();
                
                // 計算損益平衡點
                calculateBreakeven();
                
                // 恢復按鈕
                btn.innerHTML = originalText;
                btn.classList.remove('animation-bounce');
                
                // 切換到結果頁面
                if (autoSwitch && document.querySelector('.tabs .tab.active').textContent.includes('參數')) {
                    switchTab(2);
                }
            }, 800);
        }
        
        // 全局圖表變數
        let cashValueChartInstance = null;
        let returnChartInstance = null;
        
        function updateCharts() {
            // 銷毀舊的圖表實例
            if (cashValueChartInstance) {
                cashValueChartInstance.destroy();
            }
            if (returnChartInstance) {
                returnChartInstance.destroy();
            }
            
            // 重新計算數據以獲取最新值
            const cashValueData = calculateCashValue();
            const oldVal = cashValueData.oldCashValue;
            const newVal = cashValueData.newCashValue; // Guaranteed
            const newValDeclared = cashValueData.newCashValueDeclared; // Declared
            const totalPrem = cashValueData.newTotalPremiumNTD;
            const oldGuaranteedRate = cashValueData.oldGuaranteedRate;
            const newGuaranteedRate = cashValueData.newGuaranteedRate;

            // 模擬新保單現金價值成長曲線 (更真實的曲線：前低後高)
            // 使用預定利率計算保證價值曲線
            const newPolicyCurve = [
                totalPrem * 0.3,  // 目前
                totalPrem * 0.92, // 5年後 (保證價值通常較低)
                newVal * 1.0,     // 10年後 (基準點)
                newVal * Math.pow(1 + newGuaranteedRate/100, 5),    // 15年後
                newVal * Math.pow(1 + newGuaranteedRate/100, 10)    // 20年後
            ];

            // 舊保單 (已滿期，穩定成長，使用預定利率計算保證價值)
            const oldPolicyCurve = [
                oldVal,
                oldVal * Math.pow(1 + oldGuaranteedRate/100, 5),
                oldVal * Math.pow(1 + oldGuaranteedRate/100, 10),
                oldVal * Math.pow(1 + oldGuaranteedRate/100, 15),
                oldVal * Math.pow(1 + oldGuaranteedRate/100, 20)
            ];

            // 本金線 (總繳保費)
            const principalLine = Array(5).fill(totalPrem);

            // 現金價值圖表
            const ctx1 = document.getElementById('cashValueChart').getContext('2d');
            cashValueChartInstance = new Chart(ctx1, {
                type: 'bar',
                data: {
                    labels: ['目前', '5年後', '10年後', '15年後', '20年後'],
                    datasets: [
                        {
                            label: '舊保單保證價值 (NTD)',
                            data: oldPolicyCurve.map(v => Math.round(v)),
                            backgroundColor: 'rgba(102, 126, 234, 0.8)',
                            borderColor: 'rgba(102, 126, 234, 1)',
                            borderWidth: 1,
                            borderRadius: 4,
                            order: 2
                        },
                        {
                            label: '新保單保證價值 (NTD)',
                            data: newPolicyCurve.map(v => Math.round(v)),
                            backgroundColor: 'rgba(79, 172, 254, 0.8)',
                            borderColor: 'rgba(79, 172, 254, 1)',
                            borderWidth: 1,
                            borderRadius: 4,
                            order: 3
                        },
                        {
                            type: 'line',
                            label: '新保單總繳本金',
                            data: principalLine.map(v => Math.round(v)),
                            borderColor: '#ff6b6b',
                            borderWidth: 2,
                            borderDash: [5, 5],
                            pointStyle: 'line',
                            fill: false,
                            order: 1
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        title: {
                            display: true,
                            text: `現金價值成長比較 (基於預定利率: 舊${oldGuaranteedRate}% vs 新${newGuaranteedRate}%)`,
                            font: {
                                size: 16,
                                weight: 'bold'
                            }
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    return context.dataset.label + ': ' + context.raw + ' 萬';
                                }
                            }
                        },
                        subtitle: {
                            display: true,
                            text: '註：圖表顯示數值為「保證收益」，實際宣告利率可能帶來更高收益',
                            padding: {
                                bottom: 10
                            }
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            title: {
                                display: true,
                                text: '金額 (萬元)'
                            }
                        }
                    }
                }
            });
            
            // 報酬率圖表
            const ctx2 = document.getElementById('returnChart').getContext('2d');
            returnChartInstance = new Chart(ctx2, {
                type: 'line',
                data: {
                    labels: ['1年', '3年', '5年', '7年', '10年', '15年', '20年'],
                    datasets: [
                        {
                            label: '舊保單累計報酬率',
                            data: [2.5, 7.7, 13.4, 19.6, 28.0, 43.2, 60.1],
                            borderColor: 'rgba(220, 53, 69, 1)',
                            backgroundColor: 'rgba(220, 53, 69, 0.1)',
                            borderWidth: 3,
                            fill: true,
                            tension: 0.3
                        },
                        {
                            label: '新保單累計報酬率',
                            data: [0, 0, 12.5, 26.8, 48.3, 93.5, 132.8],
                            borderColor: 'rgba(40, 167, 69, 1)',
                            backgroundColor: 'rgba(40, 167, 69, 0.1)',
                            borderWidth: 3,
                            fill: true,
                            tension: 0.3
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        title: {
                            display: true,
                            text: '累計報酬率比較 (%)',
                            font: {
                                size: 16,
                                weight: 'bold'
                            }
                        },
                        legend: {
                            position: 'top',
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            grid: {
                                color: 'rgba(0, 0, 0, 0.1)'
                            },
                            title: {
                                display: true,
                                text: '累計報酬率 (%)'
                            }
                        },
                        x: {
                            grid: {
                                color: 'rgba(0, 0, 0, 0.05)'
                            }
                        }
                    }
                }
            });
        }
        
        function generateReport() {
            alert('📋 完整分析報告生成中...\n\n包含以下專業內容：\n• 新舊保單現金價值詳細比較\n• IRR報酬率分析與情境模擬\n• 保障功能CP值評估\n• 風險管理建議\n• 個人化轉換策略\n\n報告將在3秒後下載...');
            
            setTimeout(() => {
                alert('✅ 報告已生成！\n\n專業建議總結：\n1. 保留舊保單現有價值\n2. 用新增資金加買新保單\n3. 善用附加醫療保障功能\n4. 避免全解約轉換風險\n\n請向客戶完整說明所有數據，確保透明銷售。');
            }, 3000);
        }
        
        function printAllPages() {
            // 顯示所有分頁內容以便列印
            const allTabContents = document.querySelectorAll('.tab-content');
            allTabContents.forEach(content => {
                content.style.display = 'block';
                content.classList.add('active');
            });
            
            // 隱藏標籤欄
            const tabs = document.querySelector('.tabs');
            tabs.style.display = 'none';
            
            // 開始列印
            setTimeout(() => {
                window.print();
                
                // 列印完成後恢復原狀
                setTimeout(() => {
                    tabs.style.display = 'flex';
                    allTabContents.forEach((content, index) => {
                        if (index > 0) {
                            content.style.display = 'none';
                            content.classList.remove('active');
                        }
                    });
                }, 1000);
            }, 300);
        }
        
        function exportToPDF() {
            // 準備PDF匯出
            alert('📄 正在準備PDF匯出...\n\n1. 確保所有圖表已載入完成\n2. 建議使用Chrome瀏覽器效果最佳\n3. 在列印對話框選擇「另存為PDF」\n\n即將開啟列印對話框...');
            
            setTimeout(() => {
                printAllPages();
            }, 1500);
        }
        
        function showAllContent() {
            const allTabContents = document.querySelectorAll('.tab-content');
            const tabs = document.querySelector('.tabs');
            const isShowingAll = document.body.classList.contains('show-all-content');
            
            if (isShowingAll) {
                // 恢復正常模式
                document.body.classList.remove('show-all-content');
                tabs.style.display = 'flex';
                
                allTabContents.forEach((content, index) => {
                    if (index > 0) {
                        content.style.display = 'none';
                        content.classList.remove('active');
                    }
                });
                
                // 顯示第一個分頁
                allTabContents[0].style.display = 'block';
                allTabContents[0].classList.add('active');
                
                // 更新按鈕文字
                event.target.innerHTML = '<i>📱</i>iPad完整預覽';
                
                window.scrollTo({ top: 0, behavior: 'smooth' });
            } else {
                // 顯示所有內容
                document.body.classList.add('show-all-content');
                tabs.style.display = 'none';
                
                allTabContents.forEach(content => {
                    content.style.display = 'block';
                    content.classList.add('active');
                });
                
                // 確保圖表重新繪製
                setTimeout(updateCharts, 300);
                
                // 更新按鈕文字
                event.target.innerHTML = '<i>📑</i>返回分頁模式';
                
                window.scrollTo({ top: 0, behavior: 'smooth' });
            }
        }
        
        // iPad觸控優化
        function optimizeForTouch() {
            // 增強觸控回饋
            const touchElements = document.querySelectorAll('.tab, .btn, .currency-btn');
            touchElements.forEach(element => {
                element.addEventListener('touchstart', function() {
                    this.style.transform = 'scale(0.95)';
                });
                
                element.addEventListener('touchend', function() {
                    setTimeout(() => {
                        this.style.transform = '';
                    }, 150);
                });
            });
        }
        
        // 檢查是否為iPad
        function isiPad() {
            return navigator.platform === 'MacIntel' && navigator.maxTouchPoints > 1 ||
                   navigator.userAgent.includes('iPad') ||
                   (navigator.platform === 'iPad');
        }
        
        // 優化頁面切換
        function switchTab(tabNumber) {
            // 移除所有 active class
            document.querySelectorAll('.tab').forEach(tab => tab.classList.remove('active'));
            document.querySelectorAll('.tab-content').forEach(content => content.classList.remove('active'));
            
            // 添加 active class 到選中的 tab 和內容
            document.querySelector(`.tabs .tab:nth-child(${tabNumber})`).classList.add('active');
            document.getElementById(`tab${tabNumber}`).classList.add('active');
            
            // iPad滾動到頂部
            if (isiPad()) {
                window.scrollTo({
                    top: 0,
                    behavior: 'smooth'
                });
            }
            
            // 如果是結果頁面且還沒計算，自動計算
            if (tabNumber > 1) {
                setTimeout(() => calculateAll(), 200);
            }
        }
        
        // 防止頁面縮放
        function preventZoom() {
            document.addEventListener('gesturestart', function (e) {
                e.preventDefault();
            });
            
            document.addEventListener('gesturechange', function (e) {
                e.preventDefault();
            });
            
            document.addEventListener('gestureend', function (e) {
                e.preventDefault();
            });
        }
        
        // 列印前準備
        function preparePrintContent() {
            // 確保所有圖表已渲染
            const charts = document.querySelectorAll('canvas');
            charts.forEach(chart => {
                if (chart.style.display === 'none') {
                    chart.style.display = 'block';
                }
            });
            
            // 強制重新計算
            calculateAll();
        }
        
        // 初始化
        document.addEventListener('DOMContentLoaded', function() {
            // 設定預設值
            document.getElementById('clientAge').value = '45';
            document.getElementById('oldCashValueInput').value = '150';
            document.getElementById('oldRate').value = '2.5';
            document.getElementById('oldAnnualPayout').value = '3.75';
            document.getElementById('oldCoverage').value = '200';
            document.getElementById('newPremium').value = '30';
            document.getElementById('newRate').value = '4.2';
            document.getElementById('newCoverageMultiple').value = '2.0';
            document.getElementById('cancerBenefit').value = '100';
            document.getElementById('criticalIllness').value = '60';
            document.getElementById('longTermCare').value = '3';
            
            // iPad 專用優化
            if (isiPad()) {
                optimizeForTouch();
                preventZoom();
                
                // 添加iPad專用CSS類
                document.body.classList.add('ipad-optimized');
            }
            
            // 首次計算
            setTimeout(() => calculateAll(false), 500);
            
            // 添加自動計算年配息的事件監聽器
            document.getElementById('oldCashValueInput').addEventListener('input', updateOldPayout);
            document.getElementById('oldRate').addEventListener('input', updateOldPayout);
            
            // 列印事件監聽
            window.addEventListener('beforeprint', preparePrintContent);
        });
        
        // 自動計算舊保單年配息
        function updateOldPayout() {
            const oldCashValue = parseFloat(document.getElementById('oldCashValueInput').value) || 0;
            const oldRate = parseFloat(document.getElementById('oldRate').value) || 0;
            const calculatedPayout = oldCashValue * (oldRate / 100);
            document.getElementById('oldAnnualPayout').value = calculatedPayout.toFixed(2);
        }
        
        // 輸入驗證函數
        function validateInput(input, min, max, type) {
            const value = parseFloat(input.value);
            const errorElement = document.getElementById(`error-${input.id}`);
            
            if (!value || isNaN(value)) {
                input.classList.add('input-error');
                input.classList.remove('input-valid');
                if (errorElement) {
                    errorElement.textContent = '請輸入有效的數值';
                    errorElement.classList.add('show');
                }
                return false;
            }
            
            if (value < min || value > max) {
                input.classList.add('input-error');
                input.classList.remove('input-valid');
                if (errorElement) {
                    errorElement.textContent = `數值應在 ${min} 到 ${max} 之間`;
                    errorElement.classList.add('show');
                }
                return false;
            }
            
            input.classList.remove('input-error');
            input.classList.add('input-valid');
            if (errorElement) {
                errorElement.classList.remove('show');
            }
            return true;
        }
        
        // 情境切換函數
        function switchScenario(scenario) {
            currentScenario = scenario;
            
            // 更新按鈕狀態
            document.querySelectorAll('.scenario-btn').forEach(btn => {
                btn.classList.remove('active');
            });
            event.target.closest('.scenario-btn').classList.add('active');
            
            // 計算不同情境下的數據
            calculateScenario(scenario);
        }
        
        // 情境計算函數
        function calculateScenario(scenario) {
            const newPremium = parseFloat(document.getElementById('newPremium').value);
            const newRate = parseFloat(document.getElementById('newRate').value);
            const policyTerm = parseInt(document.getElementById('policyTerm').value);
            
            let adjustedRate = newRate;
            let riskLevel = '中等';
            
            switch(scenario) {
                case 'optimistic':
                    adjustedRate = newRate + 0.5;
                    riskLevel = '較低';
                    break;
                case 'conservative':
                    adjustedRate = newRate - 0.5;
                    riskLevel = '中高';
                    break;
                case 'worst':
                    adjustedRate = newRate - 1.5;
                    riskLevel = '高';
                    break;
                default:
                    riskLevel = '中等';
            }
            
            const totalPremium = newPremium * policyTerm;
            const value10yr = totalPremium * Math.pow(1 + adjustedRate/100, 10);
            const value20yr = totalPremium * Math.pow(1 + adjustedRate/100, 20);
            
            // 更新顯示
            document.getElementById('scenario10yr').textContent = `${value10yr.toFixed(1)}萬元`;
            document.getElementById('scenario20yr').textContent = `${value20yr.toFixed(1)}萬元`;
            document.getElementById('scenarioIRR').textContent = `${adjustedRate.toFixed(2)}%`;
            document.getElementById('scenarioRisk').textContent = riskLevel;
            
            // 更新情境圖表
            updateScenarioChart();
        }
        
        // 更新情境圖表
        function updateScenarioChart() {
            if (scenarioChartInstance) {
                scenarioChartInstance.destroy();
            }
            
            const ctx = document.getElementById('scenarioChart').getContext('2d');
            const newRate = parseFloat(document.getElementById('newRate').value);
            const newPremium = parseFloat(document.getElementById('newPremium').value);
            const policyTerm = parseInt(document.getElementById('policyTerm').value);
            const totalPremium = newPremium * policyTerm;
            
            // 計算四種情境的數據
            const years = [5, 10, 15, 20];
            const scenarios = {
                current: years.map(y => totalPremium * Math.pow(1 + newRate/100, y)),
                optimistic: years.map(y => totalPremium * Math.pow(1 + (newRate + 0.5)/100, y)),
                conservative: years.map(y => totalPremium * Math.pow(1 + (newRate - 0.5)/100, y)),
                worst: years.map(y => totalPremium * Math.pow(1 + (newRate - 1.5)/100, y))
            };
            
            scenarioChartInstance = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: ['5年', '10年', '15年', '20年'],
                    datasets: [
                        {
                            label: '當前情境',
                            data: scenarios.current,
                            borderColor: 'rgba(102, 126, 234, 1)',
                            backgroundColor: 'rgba(102, 126, 234, 0.1)',
                            borderWidth: 3,
                            tension: 0.3
                        },
                        {
                            label: '樂觀情境',
                            data: scenarios.optimistic,
                            borderColor: 'rgba(40, 167, 69, 1)',
                            backgroundColor: 'rgba(40, 167, 69, 0.1)',
                            borderWidth: 2,
                            borderDash: [5, 5],
                            tension: 0.3
                        },
                        {
                            label: '保守情境',
                            data: scenarios.conservative,
                            borderColor: 'rgba(255, 193, 7, 1)',
                            backgroundColor: 'rgba(255, 193, 7, 0.1)',
                            borderWidth: 2,
                            borderDash: [5, 5],
                            tension: 0.3
                        },
                        {
                            label: '最差情境',
                            data: scenarios.worst,
                            borderColor: 'rgba(220, 53, 69, 1)',
                            backgroundColor: 'rgba(220, 53, 69, 0.1)',
                            borderWidth: 2,
                            borderDash: [10, 5],
                            tension: 0.3
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        title: {
                            display: true,
                            text: '不同情境下的保單價值預測',
                            font: { size: 16, weight: 'bold' }
                        },
                        legend: { position: 'top' }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            title: { display: true, text: '價值（萬元）' }
                        }
                    }
                }
            });
        }
        
        // 更新雷達圖
        function updateRadarChart() {
            if (radarChartInstance) {
                radarChartInstance.destroy();
            }
            
            const ctx = document.getElementById('radarChart').getContext('2d');
            const clientAge = parseInt(document.getElementById('clientAge').value);
            
            // 計算各維度分數（0-100）
            const oldScores = {
                收益率: 40,  // 2.5%利率
                保障額度: 50,  // 基本壽險
                流動性: 100,  // 已滿期，隨時可領
                穩定性: 90,  // 保證利率高
                附加價值: 20   // 無附加功能
            };
            
            const newScores = {
                收益率: 70,  // 4.2%利率
                保障額度: 80,  // 更高保障倍數
                流動性: 30,  // 前5年解約虧損大
                穩定性: 60,  // 利率非保證
                附加價值: 90   // 附加醫療保障
            };
            
            radarChartInstance = new Chart(ctx, {
                type: 'radar',
                data: {
                    labels: ['收益率', '保障額度', '流動性', '穩定性', '附加價值'],
                    datasets: [
                        {
                            label: '舊保單',
                            data: Object.values(oldScores),
                            borderColor: 'rgba(220, 53, 69, 1)',
                            backgroundColor: 'rgba(220, 53, 69, 0.2)',
                            borderWidth: 2
                        },
                        {
                            label: '新保單',
                            data: Object.values(newScores),
                            borderColor: 'rgba(40, 167, 69, 1)',
                            backgroundColor: 'rgba(40, 167, 69, 0.2)',
                            borderWidth: 2
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        r: {
                            beginAtZero: true,
                            max: 100,
                            ticks: { stepSize: 20 }
                        }
                    },
                    plugins: {
                        legend: { position: 'top' }
                    }
                }
            });
        }
        
        // AI智能建議生成
        function generateAISuggestion() {
            const clientAge = parseInt(document.getElementById('clientAge').value);
            const oldCashValue = parseFloat(document.getElementById('oldCashValueInput').value);
            const oldRate = parseFloat(document.getElementById('oldRate').value);
            const newRate = parseFloat(document.getElementById('newRate').value);
            const newPremium = parseFloat(document.getElementById('newPremium').value);
            
            let suggestion = '';
            
            // 根據年齡給建議
            if (clientAge < 40) {
                suggestion += `✅ <strong>建議考慮加買新保單</strong><br>您目前${clientAge}歲，時間是最大的優勢。新保單的4.2%利率在長期複利效果下，20年後可累積可觀資產。`;
            } else if (clientAge < 55) {
                suggestion += `⚖️ <strong>建議部分配置新保單</strong><br>您目前${clientAge}歲，建議保留舊保單60-70%資金，用30-40%購買新保單分散風險。`;
            } else {
                suggestion += `⚠️ <strong>建議保留舊保單為主</strong><br>您目前${clientAge}歲，舊保單的穩定性和流動性更重要，不建議大幅調整。`;
            }
            
            suggestion += '<br><br>';
            
            // 根據利差給建議
            const rateDiff = newRate - oldRate;
            if (rateDiff > 1.5) {
                suggestion += `💰 <strong>利率優勢明顯</strong>（差距${rateDiff.toFixed(1)}%）<br>新保單利率優勢顯著，適合長期持有。但需注意這是<mark>非保證利率</mark>，可能調降。`;
            } else if (rateDiff > 0.5) {
                suggestion += `📊 <strong>利率有一定優勢</strong>（差距${rateDiff.toFixed(1)}%）<br>新保單有利率優勢，但需評估附加價值是否符合需求。`;
            } else {
                suggestion += `⚠️ <strong>利率優勢不明顯</strong>（差距${rateDiff.toFixed(1)}%）<br>單純從利率角度，轉換優勢有限，需考慮其他因素。`;
            }
            
            suggestion += '<br><br>';
            
            // 現金價值評估
            if (oldCashValue > 300) {
                suggestion += `💎 <strong>現金價值高</strong>（${oldCashValue}萬元）<br><mark>強烈不建議全額解約</mark>。這是您多年累積的資產，解約後重新累積需要很長時間。`;
            } else if (oldCashValue > 100) {
                suggestion += `💵 <strong>現金價值中等</strong>（${oldCashValue}萬元）<br>建議保留大部分資金在舊保單，用小額購買新保單測試。`;
            }
            
            suggestion += '<br><br>';
            suggestion += `<strong>🎯 最終建議：</strong>採用「<strong>雙保單策略</strong>」- 保留舊保單${oldCashValue}萬元（穩健收益），另以新資金${newPremium}萬元/年購買新保單（成長潛力+醫療保障）。`;
            
            document.getElementById('aiSuggestion').innerHTML = suggestion;
        }
        
        // 計算損益平衡點
        function calculateBreakeven() {
            const oldCashValue = parseFloat(document.getElementById('oldCashValueInput').value);
            const oldRate = parseFloat(document.getElementById('oldRate').value);
            const newPremium = parseFloat(document.getElementById('newPremium').value);
            const newRate = parseFloat(document.getElementById('newRate').value);
            const policyTerm = parseInt(document.getElementById('policyTerm').value);
            
            const newTotalPremium = newPremium * policyTerm;
            
            // 簡化計算：找出新保單價值超過舊保單的年份
            let breakevenYear = 0;
            for (let year = 1; year <= 30; year++) {
                const oldValue = oldCashValue * Math.pow(1 + oldRate/100, year);
                const newValue = newTotalPremium * Math.pow(1 + newRate/100, year);
                
                if (newValue > oldValue) {
                    breakevenYear = year;
                    break;
                }
            }
            
            if (breakevenYear === 0) {
                document.getElementById('breakevenYear').textContent = '30年+';
            } else {
                document.getElementById('breakevenYear').textContent = `${breakevenYear}年`;
            }
        }
    </script>
</body>
</html>
