
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <!-- 鎖定畫面比例，防止 iOS 點擊輸入框時畫面亂放大 -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>水電工程估價單</title>
  
  <!-- 載入 Tailwind CSS 進行排版 -->
  <script src="https://cdn.tailwindcss.com"></script>
  
  <!-- 載入 React 核心庫 -->
  <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  
  <!-- 載入 Babel 編譯器 (讓瀏覽器能讀懂 JSX) -->
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

  <style>
    /* 隱藏數字輸入框的上下箭頭 */
    input[type=number]::-webkit-inner-spin-button, 
    input[type=number]::-webkit-outer-spin-button { 
      -webkit-appearance: none; 
      margin: 0; 
    }
    input[type=number] {
      -moz-appearance: textfield;
    }
    
    /* 確保下拉選單在手機上沒有多餘的 padding */
    select {
      background-position: right 2px center !important;
      padding-right: 12px !important;
    }

    /* 列印時的版面優化 - 確保完全適應 A4 且不分頁切斷 */
    @media print {
      @page { margin: 10mm; size: A4 portrait; }
      body { 
        margin: 0; 
        background-color: white !important; 
        -webkit-print-color-adjust: exact; 
        print-color-adjust: exact; 
      }
      /* 防止表格列在跨頁時被切斷 */
      tr { page-break-inside: avoid; break-inside: avoid; }
      /* 強制外層容器滿版，由瀏覽器自行縮放以適應 A4 */
      .print-container { max-width: 100% !important; width: 100% !important; margin: 0 !important; padding: 0 !important; box-shadow: none !important; border: none !important; }
    }
  </style>
</head>
<body class="bg-slate-100 text-slate-800">

  <!-- React 渲染的目標區塊 -->
  <div id="root">
    <div class="min-h-screen flex items-center justify-center text-slate-500 font-bold text-xl">
      系統載入中，請稍候...
    </div>
  </div>

  <!-- 應用程式邏輯 -->
  <script type="text/babel">
    const { useState, useEffect } = React;

    // --- 圖示元件 ---
    const Mail = ({ size=24, className="" }) => <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"></path><polyline points="22,6 12,13 2,6"></polyline></svg>;
    const Plus = ({ size=24, className="" }) => <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><line x1="12" y1="5" x2="12" y2="19"></line><line x1="5" y1="12" x2="19" y2="12"></line></svg>;
    const Trash2 = ({ size=24, className="" }) => <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path><line x1="10" y1="11" x2="10" y2="17"></line><line x1="14" y1="11" x2="14" y2="17"></line></svg>;
    const X = ({ size=24, className="" }) => <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>;
    const FileText = ({ className="" }) => <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path><polyline points="14 2 14 8 20 8"></polyline><line x1="16" y1="13" x2="8" y2="13"></line><line x1="16" y1="17" x2="8" y2="17"></line><polyline points="10 9 9 9 8 9"></polyline></svg>;
    const Send = ({ className="" }) => <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><line x1="22" y1="2" x2="11" y2="13"></line><polygon points="22 2 15 22 11 13 2 9 22 2"></polygon></svg>;
    const RotateCcw = ({ size=24, className="" }) => <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><polyline points="1 4 1 10 7 10"></polyline><path d="M3.51 15a9 9 0 1 0 2.13-9.36L1 10"></path></svg>;

    // --- 預設選項資料 ---
    const CATEGORY_OPTIONS = ['燈具及電力工程', '專用迴路及插座', '消防系統', '給排水及衛浴工程', '基礎工程'];
    const ITEM_OPTIONS = [
      '開關出線口', '崁燈出線口', '陽台&主燈&變壓器出線口', '燈條出線口', 
      '崁燈&主燈安裝費', '燈條安裝費', '崁燈開孔處理', '冷氣專用迴路', 
      '廚房/陽台/冰箱專用迴路', '電器櫃專用迴路', '暖風機專用迴路', 
      '各空間插座迴路', '燈具控制迴路', '插座出口安裝', '電視/電話/網路出線口', 
      '消防感知器', '消防灑水頭新增/移位', '消防灑水頭長短修改', '對講機系統移位配管', 
      '熱水管出口', '冷水管出口', '排水管路出口', '馬桶專用管路出口', 
      '抽風機配管工程', '衛浴設備安裝', '暖風機安裝', '全室管線打鑿工程', 
      '開關箱及盤面整理'
    ];
    const CHINESE_NUMBERS = ['一', '二', '三', '四', '五', '六', '七', '八', '九', '十', '十一', '十二'];

    function App() {
      const [headerData, setHeaderData] = useState({ ownerName: '木柵路三段張公館', ownerAddress: '' });
      const [showTax, setShowTax] = useState(false);

      const today = new Date();
      const formattedDate = `${today.getFullYear()}/${String(today.getMonth() + 1).padStart(2, '0')}/${String(today.getDate()).padStart(2, '0')}`;

      const [items, setItems] = useState([
        { id: 'c1', type: 'category', col1: '燈具及電力工程', isCustomCol1: false },
        { id: '1', type: 'item', col1: '1', col2: '開關出線口', qty: 40, unit: '口', price: 1500, remark: '', isCustomCol2: false },
        { id: '2', type: 'item', col1: '2', col2: '崁燈出線口', qty: 65, unit: '口', price: 500, remark: '', isCustomCol2: false },
        { id: '3', type: 'item', col1: '3', col2: '陽台&主燈&變壓器出線口', qty: 15, unit: '口', price: 500, remark: '', isCustomCol2: false },
        { id: '4', type: 'item', col1: '5', col2: '崁燈&主燈安裝費', qty: 1, unit: '式', price: 22000, remark: '', isCustomCol2: false },
        { id: 'c2', type: 'category', col1: '專用迴路及插座', isCustomCol1: false },
        { id: '5', type: 'item', col1: '1', col2: '冷氣專用迴路', qty: 6, unit: '迴', price: 3500, remark: '5.5mm²', isCustomCol2: false },
        { id: '6', type: 'item', col1: '2', col2: '廚房/陽台/冰箱專用迴路', qty: 2, unit: '迴', price: 3500, remark: '5.5mm²', isCustomCol2: false },
      ]);

      const [grandTotal, setGrandTotal] = useState(0);
      const [modal, setModal] = useState({ show: false, message: '', onConfirm: null });

      // 自動計算總計
      useEffect(() => {
        let total = 0;
        items.forEach(item => {
          if (item.type === 'item') {
            const qty = parseFloat(item.qty) || 0;
            const price = parseFloat(item.price) || 0;
            total += (qty * price);
          }
        });
        setGrandTotal(total);
      }, [items]);

      // 更新資料狀態 (全面修復資料更新不同步的問題)
      const handleHeaderChange = (field, value) => {
        setHeaderData(prev => ({ ...prev, [field]: value }));
      };

      const handleItemChange = (id, field, value) => {
        setItems(prev => prev.map(item => item.id === id ? { ...item, [field]: value } : item));
      };

      const deleteItem = (id) => {
        setItems(prev => prev.filter(item => item.id !== id));
      };

      const addItem = () => {
        setItems(prev => {
          const newId = Date.now().toString();
          let lastSuffix = 0;
          for (let i = prev.length - 1; i >= 0; i--) {
            if (prev[i].type === 'category') break;
            else if (prev[i].type === 'item') {
              const parts = prev[i].col1.toString().split('.');
              const suffixText = parts.length > 1 ? parts.slice(1).join('.') : prev[i].col1;
              const suffix = parseInt(suffixText, 10);
              if (!isNaN(suffix)) lastSuffix = Math.max(lastSuffix, suffix);
            }
          }
          return [...prev, { id: newId, type: 'item', col1: (lastSuffix + 1).toString(), col2: '', qty: 1, unit: '式', price: 0, remark: '', isCustomCol2: false }];
        });
      };

      const addCategory = () => {
        setItems(prev => [...prev, { id: Date.now().toString(), type: 'category', col1: CATEGORY_OPTIONS[0], isCustomCol1: false }]);
      };

      const formatNumber = (num) => Number(num).toLocaleString('en-US');

      const handleExportAndEmail = () => {
        setModal({
          show: true,
          message: '系統即將為您產生精美報價單。\n\n請在接下來的列印視窗中選擇「另存為 PDF」。儲存完成後，系統會自動幫您開啟 Email 郵件草稿，請將剛存好的 PDF 夾帶至郵件中即可寄出。',
          onConfirm: () => {
            setModal({ show: false, message: '', onConfirm: null });
            setTimeout(() => {
              window.print();
              setTimeout(() => {
                const finalTotal = showTax ? Math.round(grandTotal * 1.08) : grandTotal;
                const subject = encodeURIComponent(`【報價單】${headerData.ownerName} - 岳鼎水電工程`);
                const body = encodeURIComponent(`您好，\n\n附件為「${headerData.ownerName}」的水電工程報價單，總金額為 NT$ ${formatNumber(finalTotal)}。\n\n請查閱附件 PDF 檔案，如有任何問題歡迎隨時聯繫。\n\n聯絡人：廖家緯 (0988676742)\n\n謝謝！`);
                window.location.href = `mailto:?subject=${subject}&body=${body}`;
              }, 1000);
            }, 300);
          }
        });
      };

      // 共用樣式：讓手機版字體極小(11px)、邊距極小(p-0.5)，確保塞得下 100% 螢幕
      const inputClass = "w-full bg-transparent outline-none p-0.5 sm:p-2 hover:bg-white focus:bg-white focus:ring-1 focus:ring-blue-300 transition-colors text-[11px] sm:text-[15px]";
      const tdClass = "border border-slate-400 p-0 sm:p-1";
      const thClass = "border border-slate-400 bg-[#b4c6e7] p-1 sm:p-2 text-center font-bold text-[11px] sm:text-[15px] text-slate-800";
      
      let currentCatIndex = 0;

      return (
        <div className="min-h-screen bg-slate-200 py-2 sm:py-8 px-1 sm:px-8 pb-32 print:p-0 print:bg-white font-sans">
          
          {modal.show && (
            <div className="fixed inset-0 bg-slate-900/60 flex items-center justify-center z-50 print:hidden backdrop-blur-sm">
              <div className="bg-white p-6 rounded-2xl shadow-2xl max-w-md w-full mx-4 border-t-4 border-blue-600">
                <h3 className="text-xl font-bold mb-4 flex items-center text-blue-800">
                  <FileText className="mr-2 text-blue-600" />
                  轉存 PDF 並發送 Email
                </h3>
                <p className="text-slate-600 whitespace-pre-line leading-relaxed mb-6">{modal.message}</p>
                <div className="flex justify-end gap-3">
                  <button onClick={() => setModal({ show: false, message: '', onConfirm: null })} className="px-4 py-2 text-slate-600 bg-slate-100 rounded-lg hover:bg-slate-200 transition font-medium">取消</button>
                  <button onClick={modal.onConfirm} className="px-4 py-2 text-white bg-blue-600 rounded-lg hover:bg-blue-700 flex items-center transition shadow-md font-medium">
                    <Send className="w-4 h-4 mr-2" />
                    了解並繼續
                  </button>
                </div>
              </div>
            </div>
          )}

          <div className="print-container max-w-5xl mx-auto bg-white shadow-xl rounded-sm sm:rounded-lg p-2 sm:p-12">
            
            {/* 標題 (還原 Excel 紅色粗體字) */}
            <h1 className="text-xl sm:text-3xl font-black text-red-600 text-center mb-3 sm:mb-6 tracking-widest">
              水電工程維修/翻修報價單
            </h1>

            {/* 表頭資訊 - 模仿 Excel 兩欄式緊湊排版 */}
            <div className="grid grid-cols-2 gap-x-2 gap-y-1 mb-2 sm:mb-4 text-[11px] sm:text-[15px] border border-slate-400 p-1 sm:p-2 bg-slate-50">
              <div className="flex items-center">
                <span className="font-bold whitespace-nowrap mr-1 w-14 sm:w-20">工程名稱：</span>
                <input type="text" className="w-full bg-transparent outline-none focus:bg-white px-1 font-bold text-blue-900 border-b border-dashed border-slate-300" value={headerData.ownerName} onChange={(e) => handleHeaderChange('ownerName', e.target.value)} placeholder="請輸入業主名稱" />
              </div>
              <div className="flex items-center">
                <span className="font-bold whitespace-nowrap mr-1 w-14 sm:w-20">公司名稱：</span>
                <span className="w-full px-1">岳鼎水電工程</span>
              </div>
              
              <div className="flex items-center">
                <span className="font-bold whitespace-nowrap mr-1 w-14 sm:w-20">報價日期：</span>
                <span className="w-full px-1">{formattedDate}</span>
              </div>
              <div className="flex items-center">
                <span className="font-bold whitespace-nowrap mr-1 w-14 sm:w-20">公司地址：</span>
                <span className="w-full px-1 truncate">新北市蘆洲區光華路150巷13弄1號1樓</span>
              </div>

              <div className="flex items-center">
                <span className="font-bold whitespace-nowrap mr-1 w-14 sm:w-20">業主地址：</span>
                <input type="text" className="w-full bg-transparent outline-none focus:bg-white px-1 border-b border-dashed border-slate-300" value={headerData.ownerAddress} onChange={(e) => handleHeaderChange('ownerAddress', e.target.value)} placeholder="請輸入地址" />
              </div>
              <div className="flex items-center">
                <span className="font-bold whitespace-nowrap mr-1 w-14 sm:w-20">聯絡人：</span>
                <span className="w-full px-1">廖家緯 (0988676742)</span>
              </div>
            </div>

            {/* 表格區塊：強制 width=100%, 採用 table-fixed, 嚴格分配百分比寬度 */}
            <div className="w-full border-2 border-slate-400 print:border-slate-800">
              <table className="w-full table-fixed border-collapse">
                <thead>
                  <tr>
                    <th className={`${thClass} w-[8%]`}>編號</th>
                    <th className={`${thClass} w-[34%] text-left`}>項目名稱及規格規範</th>
                    <th className={`${thClass} w-[9%]`}>數量</th>
                    <th className={`${thClass} w-[9%]`}>單位</th>
                    <th className={`${thClass} w-[15%]`}>單價</th>
                    <th className={`${thClass} w-[16%]`}>複價</th>
                    <th className={`${thClass} w-[9%]`}>備註</th>
                  </tr>
                </thead>
                <tbody>
                  {items.map((item) => {
                    // --- 分類標題列 ---
                    if (item.type === 'category') {
                      currentCatIndex++;
                      const prefixText = (CHINESE_NUMBERS[currentCatIndex - 1] || currentCatIndex) + '、';
                      return (
                        <tr key={item.id} className="border-b border-slate-400 bg-slate-100 group">
                          <td colSpan="7" className="p-0 border border-slate-400 relative">
                            <div className="flex items-center px-1">
                              <span className="font-bold text-slate-800 text-[11px] sm:text-[15px] whitespace-nowrap shrink-0">{prefixText}</span>
                              {item.isCustomCol1 ? (
                                <div className="flex items-center w-full">
                                  <input type="text" className={`${inputClass} font-bold text-slate-800 ml-1`} value={item.col1} onChange={(e) => handleItemChange(item.id, 'col1', e.target.value)} placeholder="自訂名稱..." autoFocus />
                                  <button onClick={() => handleItemChange(item.id, 'isCustomCol1', false)} className="p-1 text-slate-400 print:hidden shrink-0"><RotateCcw size={12} /></button>
                                </div>
                              ) : (
                                <div className="flex items-center w-full">
                                  <select 
                                    className="w-full p-0.5 sm:p-2 bg-transparent outline-none font-bold text-slate-800 text-[11px] sm:text-[15px] appearance-none cursor-pointer print:appearance-none hover:bg-slate-200 transition" 
                                    value={item.col1} 
                                    onChange={(e) => { 
                                      if (e.target.value === 'CUSTOM') { 
                                        // 修復自訂切換邏輯
                                        setItems(prev => prev.map(i => i.id === item.id ? { ...i, isCustomCol1: true, col1: '' } : i));
                                      } else {
                                        handleItemChange(item.id, 'col1', e.target.value); 
                                      }
                                    }}
                                  >
                                    <option value="" disabled>請選擇分類...</option>
                                    {CATEGORY_OPTIONS.map(opt => <option key={opt} value={opt}>{opt}</option>)}
                                    <option value="CUSTOM" className="text-blue-600">✎ 自訂輸入...</option>
                                  </select>
                                </div>
                              )}
                            </div>
                            {/* 分類刪除按鈕 */}
                            <button onClick={() => deleteItem(item.id)} className="absolute right-1 top-1/2 -translate-y-1/2 text-red-400 hover:text-red-600 p-1 opacity-0 group-hover:opacity-100 transition print:hidden bg-white rounded shadow-sm"><Trash2 size={12} /></button>
                          </td>
                        </tr>
                      );
                    }

                    // --- 一般項目列 ---
                    const subtotal = (parseFloat(item.qty) || 0) * (parseFloat(item.price) || 0);
                    const parts = (item.col1 || '').toString().split('.');
                    const suffix = parts.length > 1 ? parts.slice(1).join('.') : item.col1;
                    const displayCol1 = currentCatIndex > 0 ? `${currentCatIndex}.${suffix}` : suffix;

                    return (
                      <tr key={item.id} className="border-b border-slate-400 hover:bg-blue-50 transition group">
                        <td className={tdClass}>
                          <input type="text" className={`${inputClass} text-center font-medium`} value={displayCol1} onChange={(e) => handleItemChange(item.id, 'col1', e.target.value)} />
                        </td>
                        <td className={tdClass}>
                          {item.isCustomCol2 ? (
                            <div className="flex items-center w-full overflow-hidden">
                              <input type="text" className={`${inputClass} truncate`} value={item.col2} onChange={(e) => handleItemChange(item.id, 'col2', e.target.value)} placeholder="自訂項目..." autoFocus />
                              <button onClick={() => handleItemChange(item.id, 'isCustomCol2', false)} className="p-0.5 text-slate-400 print:hidden shrink-0"><RotateCcw size={12} /></button>
                            </div>
                          ) : (
                            <select 
                              className="w-full p-0.5 sm:p-2 bg-transparent outline-none appearance-none cursor-pointer print:appearance-none hover:bg-white truncate text-[11px] sm:text-[15px]" 
                              value={item.col2} 
                              onChange={(e) => { 
                                if (e.target.value === 'CUSTOM') { 
                                  // 修復自訂切換邏輯
                                  setItems(prev => prev.map(i => i.id === item.id ? { ...i, isCustomCol2: true, col2: '' } : i));
                                } else {
                                  handleItemChange(item.id, 'col2', e.target.value); 
                                }
                              }}
                            >
                              <option value="" disabled>請選擇...</option>
                              {ITEM_OPTIONS.map(opt => <option key={opt} value={opt}>{opt}</option>)}
                              <option value="CUSTOM" className="text-blue-600">✎ 自訂輸入...</option>
                            </select>
                          )}
                        </td>
                        <td className={tdClass}>
                          <input type="number" className={`${inputClass} text-center`} value={item.qty} onChange={(e) => handleItemChange(item.id, 'qty', e.target.value)} min="0" />
                        </td>
                        <td className={tdClass}>
                          <input type="text" className={`${inputClass} text-center`} value={item.unit} onChange={(e) => handleItemChange(item.id, 'unit', e.target.value)} />
                        </td>
                        <td className={tdClass}>
                          <input type="number" className={`${inputClass} text-right`} value={item.price} onChange={(e) => handleItemChange(item.id, 'price', e.target.value)} min="0" />
                        </td>
                        <td className={tdClass}>
                          <div className="w-full text-right p-0.5 sm:p-2 font-medium text-[11px] sm:text-[15px]">
                            {formatNumber(subtotal)}
                          </div>
                        </td>
                        <td className={`${tdClass} relative`}>
                          <input type="text" className={`${inputClass} text-left text-slate-500`} value={item.remark} onChange={(e) => handleItemChange(item.id, 'remark', e.target.value)} />
                          {/* 刪除按鈕 (隱藏在備註欄右側，Hover 顯示) */}
                          <button onClick={() => deleteItem(item.id)} className="absolute right-0 top-1/2 -translate-y-1/2 text-red-500 hover:bg-red-50 p-1 opacity-0 group-hover:opacity-100 transition print:hidden z-10"><Trash2 size={14} /></button>
                        </td>
                      </tr>
                    );
                  })}
                  
                  {/* 空白列填充 (視覺效果) */}
                  <tr>
                    <td className={tdClass}><div className="h-4 sm:h-6"></div></td>
                    <td className={tdClass}></td><td className={tdClass}></td><td className={tdClass}></td><td className={tdClass}></td><td className={tdClass}></td><td className={tdClass}></td>
                  </tr>

                  {/* 總結算區塊 */}
                  <tr>
                    <td colSpan="4" className="border border-slate-400 p-1 sm:p-2 text-[10px] sm:text-sm text-slate-700 font-medium align-top" rowSpan={showTax ? 3 : 1}>
                      <ul className="list-disc pl-4 space-y-0.5 sm:space-y-1">
                        <li>付款方式：訂金 30%、進度款 40%、驗收結案 30%。</li>
                        <li>本工程不含水泥修補、油漆及磁磚復原。</li>
                        <li>保固期限：水電管線保固 1 年 (非人為損壞)。</li>
                        <li>若遇牆體結構過硬或特殊現場環境，施工費用另計。</li>
                      </ul>
                    </td>
                    <td className="border border-slate-400 p-1 sm:p-2 text-right font-bold text-slate-800 text-[11px] sm:text-[15px] align-middle">
                      {showTax ? '報價金額：' : '總計金額：'}
                    </td>
                    <td className="border border-slate-400 p-1 sm:p-2 text-right font-black text-red-600 text-[13px] sm:text-lg align-middle">
                      {formatNumber(grandTotal)}
                    </td>
                    <td className="border border-slate-400 p-1 sm:p-2 text-center text-slate-500 text-[10px] sm:text-[13px] align-middle"></td>
                  </tr>
                  
                  {showTax && (
                    <>
                      <tr>
                        <td className="border border-slate-400 p-1 sm:p-2 text-right font-bold text-slate-800 text-[11px] sm:text-[15px] align-middle">營業稅 (8%)：</td>
                        <td className="border border-slate-400 p-1 sm:p-2 text-right font-bold text-slate-800 text-[12px] sm:text-[16px] align-middle">{formatNumber(Math.round(grandTotal * 0.08))}</td>
                        <td className="border border-slate-400"></td>
                      </tr>
                      <tr>
                        <td className="border border-slate-400 p-1 sm:p-2 text-right font-black text-red-600 text-[12px] sm:text-[16px] align-middle">含稅總金額：</td>
                        <td className="border border-slate-400 p-1 sm:p-2 text-right font-black text-red-600 text-[14px] sm:text-[20px] align-middle">{formatNumber(Math.round(grandTotal * 1.08))}</td>
                        <td className="border border-slate-400"></td>
                      </tr>
                    </>
                  )}
                </tbody>
              </table>
            </div>

            <div className="mt-3 flex justify-start print:hidden">
              <button onClick={() => setShowTax(!showTax)} className={`flex items-center px-3 py-1.5 rounded text-xs sm:text-sm font-bold transition-all border ${showTax ? 'bg-red-50 text-red-600 border-red-200' : 'bg-white text-slate-600 border-slate-300 hover:bg-slate-50'}`}>
                {showTax ? <X size={14} className="mr-1" /> : <Plus size={14} className="mr-1" />}
                {showTax ? '移除 8% 營業稅' : '新增 8% 營業稅'}
              </button>
            </div>
          </div>

          {/* 右下角浮動按鈕 (加強縮小以防擋住手機畫面) */}
          <div className="fixed bottom-3 right-3 sm:bottom-6 sm:right-6 flex flex-col items-end gap-2 print:hidden z-40">
            <button onClick={addCategory} className="bg-cyan-600 hover:bg-cyan-700 text-white w-10 h-10 sm:w-auto sm:h-auto sm:px-4 sm:py-2.5 rounded-full shadow-lg flex items-center justify-center transition font-bold" title="新增分類">
              <Plus size={20} className="sm:mr-2" /><span className="hidden sm:inline text-sm">新增分類</span>
            </button>
            <button onClick={addItem} className="bg-sky-500 hover:bg-sky-600 text-white w-10 h-10 sm:w-auto sm:h-auto sm:px-4 sm:py-2.5 rounded-full shadow-lg flex items-center justify-center transition font-bold" title="新增項目">
              <Plus size={20} className="sm:mr-2" /><span className="hidden sm:inline text-sm">新增項目</span>
            </button>
            <button onClick={handleExportAndEmail} className="bg-blue-700 hover:bg-blue-800 text-white w-12 h-12 sm:w-auto sm:h-auto sm:px-5 sm:py-3.5 rounded-full shadow-xl flex items-center justify-center transition transform hover:-translate-y-1 mt-1 border-2 border-white">
              <Mail size={22} className="sm:mr-2" /><span className="hidden sm:inline font-black tracking-wide text-sm">輸出 PDF 並發送信件</span>
            </button>
          </div>

        </div>
      );
    }

    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<App />);
  </script>
</body>
</html>
