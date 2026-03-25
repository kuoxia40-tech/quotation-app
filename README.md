<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <!-- 加入 maximum-scale=1.0, user-scalable=no 防止手機 (尤其 iOS) 點擊輸入框時畫面亂放大變形 -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>水電工程報價系統</title>
  
  <!-- 載入 Tailwind CSS 進行排版 -->
  <script src="https://cdn.tailwindcss.com"></script>
  
  <!-- 載入 React 核心庫 -->
  <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  
  <!-- 載入 Babel 編譯器 (讓瀏覽器能讀懂 JSX) -->
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

  <style>
    /* 隱藏數字輸入框的上下箭頭，釋放更多寬度給數字顯示 */
    input[type=number]::-webkit-inner-spin-button, 
    input[type=number]::-webkit-outer-spin-button { 
      -webkit-appearance: none; 
      margin: 0; 
    }
    input[type=number] {
      -moz-appearance: textfield;
    }
    /* 列印時的版面優化 */
    @media print {
      @page { margin: 0; }
      body { margin: 1.6cm; background-color: white !important; }
    }
  </style>
</head>
<body class="bg-slate-100">

  <!-- React 渲染的目標區塊 -->
  <div id="root"></div>

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
    const ChevronDown = ({ size=24, className="" }) => <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><polyline points="6 9 12 15 18 9"></polyline></svg>;

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

      const handleHeaderChange = (field, value) => setHeaderData({ ...headerData, [field]: value });
      const handleItemChange = (id, field, value) => setItems(items.map(item => item.id === id ? { ...item, [field]: value } : item));
      const deleteItem = (id) => setItems(items.filter(item => item.id !== id));

      const addItem = () => {
        const newId = Date.now().toString();
        let lastSuffix = 0;
        for (let i = items.length - 1; i >= 0; i--) {
          if (items[i].type === 'category') break;
          else if (items[i].type === 'item') {
            const parts = items[i].col1.toString().split('.');
            const suffixText = parts.length > 1 ? parts.slice(1).join('.') : items[i].col1;
            const suffix = parseInt(suffixText, 10);
            if (!isNaN(suffix)) lastSuffix = Math.max(lastSuffix, suffix);
          }
        }
        setItems([...items, { id: newId, type: 'item', col1: (lastSuffix + 1).toString(), col2: '', qty: 1, unit: '式', price: 0, remark: '', isCustomCol2: false }]);
      };

      const addCategory = () => {
        const newId = Date.now().toString();
        setItems([...items, { id: newId, type: 'category', col1: CATEGORY_OPTIONS[0], isCustomCol1: false }]);
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

      const inputClass = "w-full p-2 bg-transparent outline-none rounded hover:bg-white hover:shadow-sm focus:bg-white focus:ring-2 focus:ring-blue-300 transition print:p-0 print:shadow-none print:hover:bg-transparent";
      let currentCatIndex = 0;

      return (
        <div className="min-h-screen bg-slate-100 py-6 px-2 sm:py-8 sm:px-8 pb-32 print:p-0 print:bg-white text-sm sm:text-base font-sans text-slate-800">
          
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

          <div className="max-w-5xl mx-auto bg-white shadow-xl rounded-lg p-3 sm:p-12 print:shadow-none print:p-0 print:max-w-none print:rounded-none">
            {/* 標題 */}
            <h1 className="text-2xl sm:text-4xl font-black text-blue-900 text-center mb-6 sm:mb-8 tracking-widest border-b-2 sm:border-b-4 border-blue-900 pb-3 sm:pb-4 print:border-b-2">
              水電工程維修 / 翻修報價單
            </h1>

            {/* 表頭資訊 */}
            <div className="grid grid-cols-1 sm:grid-cols-2 border-2 border-slate-300 rounded-lg mb-6 overflow-hidden print:border-slate-800">
              <div className="flex border-b sm:border-r border-slate-300 print:border-slate-800">
                <div className="bg-slate-100 text-slate-700 font-bold w-16 sm:w-20 px-2 py-2 text-xs sm:text-sm border-r border-slate-300 print:border-slate-800 flex-shrink-0 flex items-center whitespace-nowrap">公司名稱</div>
                <div className="px-2 py-2 w-full font-black text-blue-900 text-xs sm:text-sm flex items-center tracking-wider">岳鼎水電工程</div>
              </div>
              <div className="flex border-b border-slate-300 print:border-slate-800">
                <div className="bg-slate-100 text-slate-700 font-bold w-16 sm:w-20 px-2 py-2 text-xs sm:text-sm border-r border-slate-300 print:border-slate-800 flex-shrink-0 flex items-center whitespace-nowrap">業主名稱</div>
                <input type="text" className="px-2 py-2 w-full outline-none focus:bg-blue-50 transition font-bold text-slate-800 text-xs sm:text-sm" value={headerData.ownerName} onChange={(e) => handleHeaderChange('ownerName', e.target.value)} placeholder="請輸入業主名稱" />
              </div>
              <div className="flex border-b sm:border-r border-slate-300 print:border-slate-800">
                <div className="bg-slate-100 text-slate-700 font-bold w-16 sm:w-20 px-2 py-2 text-xs sm:text-sm border-r border-slate-300 print:border-slate-800 flex-shrink-0 flex items-center whitespace-nowrap">公司地址</div>
                <div className="px-2 py-2 w-full text-slate-700 text-xs sm:text-sm flex items-center font-medium">新北市蘆洲區光華路150巷13弄1號1樓</div>
              </div>
              <div className="flex border-b border-slate-300 print:border-slate-800">
                <div className="bg-slate-100 text-slate-700 font-bold w-16 sm:w-20 px-2 py-2 text-xs sm:text-sm border-r border-slate-300 print:border-slate-800 flex-shrink-0 flex items-center whitespace-nowrap">業主地址</div>
                <input type="text" className="px-2 py-2 w-full outline-none focus:bg-blue-50 transition text-slate-800 text-xs sm:text-sm" value={headerData.ownerAddress} onChange={(e) => handleHeaderChange('ownerAddress', e.target.value)} placeholder="請輸入業主地址" />
              </div>
              <div className="flex border-b sm:border-b-0 sm:border-r border-slate-300 print:border-slate-800">
                <div className="bg-slate-100 text-slate-700 font-bold w-16 sm:w-20 px-2 py-2 text-xs sm:text-sm border-r border-slate-300 print:border-slate-800 flex-shrink-0 flex items-center whitespace-nowrap">聯絡人</div>
                <div className="px-2 py-2 w-full font-bold text-slate-700 text-xs sm:text-sm flex items-center">廖家緯 (0988676742)</div>
              </div>
              <div className="flex">
                <div className="bg-slate-100 text-slate-700 font-bold w-16 sm:w-20 px-2 py-2 text-xs sm:text-sm border-r border-slate-300 print:border-slate-800 flex-shrink-0 flex items-center whitespace-nowrap">報價日期</div>
                <div className="px-2 py-2 w-full text-slate-700 text-xs sm:text-sm flex items-center font-medium">{formattedDate}</div>
              </div>
            </div>

            {/* 報價單表格區塊：加入 overflow-x-auto 與 Webkit 觸控支援，並強迫表格有最小寬度 */}
            <div className="overflow-x-auto rounded-lg border-2 border-blue-800 print:border-slate-800 w-full" style={{ WebkitOverflowScrolling: 'touch' }}>
              {/* 加入 min-w-[800px] 防止手機版擠壓，但列印時 (print:min-w-0) 允許自然縮放 */}
              <table className="w-full min-w-[850px] print:min-w-0 border-collapse">
                <thead>
                  <tr className="bg-blue-800 text-white print:bg-slate-200 print:text-black text-sm sm:text-base">
                    <th className="border-r border-blue-700 print:border-slate-800 p-2 text-center w-16 print:w-12 font-bold">編號</th>
                    <th className="border-r border-blue-700 print:border-slate-800 p-2 text-left min-w-[200px] print:min-w-0 font-bold">項目名稱及規格規範</th>
                    <th className="border-r border-blue-700 print:border-slate-800 p-2 text-center w-20 print:w-16 font-bold">數量</th>
                    <th className="border-r border-blue-700 print:border-slate-800 p-2 text-center w-16 print:w-12 font-bold">單位</th>
                    <th className="border-r border-blue-700 print:border-slate-800 p-2 text-right w-24 print:w-20 font-bold">單價</th>
                    <th className="border-r border-blue-700 print:border-slate-800 p-2 text-right w-32 print:w-24 font-bold">複價</th>
                    <th className="p-2 text-left w-32 print:w-24 relative font-bold">
                      備註說明
                      <span className="absolute right-2 top-2 print:hidden text-[10px] text-blue-300 opacity-80">操作</span>
                    </th>
                  </tr>
                </thead>
                <tbody>
                  {items.map((item) => {
                    if (item.type === 'category') {
                      currentCatIndex++;
                      const prefixText = (CHINESE_NUMBERS[currentCatIndex - 1] || currentCatIndex) + '、';
                      return (
                        <tr key={item.id} className="border-b border-slate-300 bg-blue-50 group print:bg-slate-100">
                          <td colSpan="6" className="border-r border-slate-300 p-1">
                            {item.isCustomCol1 ? (
                              <div className="flex items-center px-2">
                                <span className="font-bold text-blue-900 text-lg whitespace-nowrap shrink-0">{prefixText}</span>
                                <input type="text" className={`${inputClass} font-bold text-blue-900 text-lg ml-1`} value={item.col1} onChange={(e) => handleItemChange(item.id, 'col1', e.target.value)} placeholder="請輸入自訂分類名稱..." autoFocus />
                                <button onClick={() => handleItemChange(item.id, 'isCustomCol1', false)} className="ml-2 p-1 text-slate-400 hover:text-blue-600 print:hidden" title="切換回下拉選單"><RotateCcw size={16} /></button>
                              </div>
                            ) : (
                              <div className="flex items-center px-2 relative">
                                <span className="font-bold text-blue-900 text-lg whitespace-nowrap shrink-0">{prefixText}</span>
                                <select className="w-full p-2 bg-transparent outline-none font-bold text-blue-900 text-lg appearance-none cursor-pointer print:appearance-none hover:bg-blue-100 rounded transition ml-1" value={item.col1} onChange={(e) => { if (e.target.value === 'CUSTOM') { handleItemChange(item.id, 'isCustomCol1', true); handleItemChange(item.id, 'col1', ''); } else handleItemChange(item.id, 'col1', e.target.value); }}>
                                  <option value="" disabled>請選擇分類...</option>
                                  {CATEGORY_OPTIONS.map(opt => <option key={opt} value={opt}>{opt}</option>)}
                                  <option value="CUSTOM" className="font-bold text-blue-600">✎ 自訂輸入...</option>
                                </select>
                                <ChevronDown size={16} className="absolute right-4 top-1/2 -translate-y-1/2 text-blue-500 pointer-events-none print:hidden" />
                              </div>
                            )}
                          </td>
                          <td className="p-2 text-center align-middle bg-blue-50 print:bg-slate-100">
                            <button onClick={() => deleteItem(item.id)} className="text-red-400 hover:text-red-600 p-2 opacity-0 group-hover:opacity-100 transition print:hidden bg-white rounded-md shadow-sm" title="刪除分類"><Trash2 size={16} /></button>
                          </td>
                        </tr>
                      );
                    }

                    const subtotal = (parseFloat(item.qty) || 0) * (parseFloat(item.price) || 0);
                    const parts = (item.col1 || '').toString().split('.');
                    const suffix = parts.length > 1 ? parts.slice(1).join('.') : item.col1;
                    const displayCol1 = currentCatIndex > 0 ? `${currentCatIndex}.${suffix}` : suffix;

                    return (
                      <tr key={item.id} className="border-b border-slate-300 hover:bg-slate-50 transition group">
                        <td className="border-r border-slate-300 p-1">
                          <input type="text" className={`${inputClass} text-center`} value={displayCol1} onChange={(e) => handleItemChange(item.id, 'col1', e.target.value)} placeholder={`如: ${currentCatIndex}.1`} />
                        </td>
                        <td className="border-r border-slate-300 p-1">
                          {item.isCustomCol2 ? (
                            <div className="flex items-center">
                              <input type="text" className={inputClass} value={item.col2} onChange={(e) => handleItemChange(item.id, 'col2', e.target.value)} placeholder="請輸入自訂項目..." autoFocus />
                              <button onClick={() => handleItemChange(item.id, 'isCustomCol2', false)} className="ml-1 p-1 text-slate-400 hover:text-blue-600 print:hidden" title="切換回下拉選單"><RotateCcw size={14} /></button>
                            </div>
                          ) : (
                            <div className="relative">
                              <select className="w-full p-2 bg-transparent outline-none appearance-none cursor-pointer print:appearance-none hover:bg-slate-100 rounded transition text-slate-800" value={item.col2} onChange={(e) => { if (e.target.value === 'CUSTOM') { handleItemChange(item.id, 'isCustomCol2', true); handleItemChange(item.id, 'col2', ''); } else handleItemChange(item.id, 'col2', e.target.value); }}>
                                <option value="" disabled>請選擇項目...</option>
                                {ITEM_OPTIONS.map(opt => <option key={opt} value={opt}>{opt}</option>)}
                                <option value="CUSTOM" className="font-bold text-blue-600">✎ 自訂輸入...</option>
                              </select>
                              <ChevronDown size={14} className="absolute right-2 top-1/2 -translate-y-1/2 text-slate-400 pointer-events-none print:hidden" />
                            </div>
                          )}
                        </td>
                        <td className="border-r border-slate-300 p-1">
                          <input type="number" className={`${inputClass} text-center`} value={item.qty} onChange={(e) => handleItemChange(item.id, 'qty', e.target.value)} min="0" />
                        </td>
                        <td className="border-r border-slate-300 p-1">
                          <input type="text" className={`${inputClass} text-center`} value={item.unit} onChange={(e) => handleItemChange(item.id, 'unit', e.target.value)} />
                        </td>
                        <td className="border-r border-slate-300 p-1">
                          <input type="number" className={`${inputClass} text-right`} value={item.price} onChange={(e) => handleItemChange(item.id, 'price', e.target.value)} min="0" />
                        </td>
                        <td className="border-r border-slate-300 p-3 text-right font-medium text-slate-800">
                          {formatNumber(subtotal)}
                        </td>
                        <td className="p-1 flex items-center justify-between h-full">
                          <input type="text" className={`${inputClass} text-sm text-slate-600`} value={item.remark} onChange={(e) => handleItemChange(item.id, 'remark', e.target.value)} placeholder="備註..." />
                          <button onClick={() => deleteItem(item.id)} className="text-red-400 hover:text-red-600 p-2 opacity-0 group-hover:opacity-100 transition print:hidden bg-slate-100 rounded-md ml-1" title="刪除項目"><Trash2 size={16} /></button>
                        </td>
                      </tr>
                    );
                  })}
                  
                  <tr className="border-t-4 border-blue-800 bg-slate-50 print:border-slate-800 print:bg-transparent">
                    <td colSpan="5" className="border-r border-slate-300 p-3 text-right font-bold text-blue-900 tracking-wider text-sm sm:text-base">
                      {showTax ? '報價金額：' : '總計金額：'}
                    </td>
                    <td className="border-r border-slate-300 p-3 text-right font-black text-red-600 text-lg sm:text-xl tracking-wider">
                      {formatNumber(grandTotal)}
                    </td>
                    <td className="p-3 text-slate-500 text-xs sm:text-sm font-medium flex items-center justify-center">(新台幣)</td>
                  </tr>
                  
                  {showTax && (
                    <>
                      <tr className="bg-slate-50 print:bg-transparent border-t border-slate-200 print:border-slate-800">
                        <td colSpan="5" className="border-r border-slate-300 p-2 sm:p-3 text-right font-bold text-slate-700 tracking-wider text-sm sm:text-base">營業稅 (8%)：</td>
                        <td className="border-r border-slate-300 p-2 sm:p-3 text-right font-bold text-slate-700 text-base sm:text-lg tracking-wider">{formatNumber(Math.round(grandTotal * 0.08))}</td>
                        <td className="p-2 sm:p-3 text-slate-500 text-xs sm:text-sm font-medium flex items-center justify-center"></td>
                      </tr>
                      <tr className="border-t-2 border-slate-400 bg-blue-100 print:border-slate-800 print:bg-transparent">
                        <td colSpan="5" className="border-r border-slate-300 p-3 text-right font-black text-blue-900 tracking-wider text-base sm:text-lg">含稅總金額：</td>
                        <td className="border-r border-slate-300 p-3 text-right font-black text-red-600 text-xl sm:text-2xl tracking-wider">{formatNumber(Math.round(grandTotal * 1.08))}</td>
                        <td className="p-3 text-slate-500 text-xs sm:text-sm font-medium flex items-center justify-center">(新台幣)</td>
                      </tr>
                    </>
                  )}
                </tbody>
              </table>
            </div>

            <div className="mt-4 flex justify-start print:hidden">
              <button onClick={() => setShowTax(!showTax)} className={`flex items-center px-4 py-2 rounded-lg font-bold transition-all shadow-sm ${showTax ? 'bg-red-50 text-red-600 hover:bg-red-100' : 'bg-slate-100 text-slate-600 hover:bg-slate-200'}`}>
                {showTax ? <X size={18} className="mr-2" /> : <Plus size={18} className="mr-2" />}
                {showTax ? '移除 8% 營業稅' : '新增 8% 營業稅'}
              </button>
            </div>

            <div className="mt-8 border-2 border-slate-300 rounded-lg p-4 sm:p-6 text-sm sm:text-base leading-loose text-slate-700 bg-slate-50 print:border-slate-800 print:bg-transparent">
              <ul className="list-disc pl-5 sm:pl-6 space-y-2 font-medium">
                <li>付款方式：訂金 <span className="text-blue-700">30%</span>、進度款 <span className="text-blue-700">40%</span>、驗收結案 <span className="text-blue-700">30%</span>。</li>
                <li>本工程不含水泥修補、油漆及磁磚復原。</li>
                <li>保固期限：水電管線保固 <span className="text-blue-700">1 年</span> (非人為損壞)。</li>
                <li>若遇牆體結構過硬或特殊現場環境，施工費用另計。</li>
              </ul>
            </div>
          </div>

          {/* 右下角浮動按鈕 */}
          <div className="fixed bottom-4 right-4 sm:bottom-6 sm:right-6 flex flex-col items-end gap-2 sm:gap-3 print:hidden z-40">
            <button onClick={addCategory} className="bg-cyan-600 hover:bg-cyan-700 text-white w-12 h-12 sm:w-auto sm:h-auto sm:px-5 sm:py-3 rounded-full shadow-lg flex items-center justify-center transition font-bold" title="新增分類標題">
              <Plus size={24} className="sm:mr-2" /><span className="hidden sm:inline">新增分類</span>
            </button>
            <button onClick={addItem} className="bg-sky-500 hover:bg-sky-600 text-white w-12 h-12 sm:w-auto sm:h-auto sm:px-5 sm:py-3 rounded-full shadow-lg flex items-center justify-center transition font-bold" title="新增工程項目">
              <Plus size={24} className="sm:mr-2" /><span className="hidden sm:inline">新增項目</span>
            </button>
            <button onClick={handleExportAndEmail} className="bg-blue-700 hover:bg-blue-800 text-white w-14 h-14 sm:w-auto sm:h-auto sm:px-6 sm:py-4 rounded-full shadow-xl flex items-center justify-center transition transform hover:-translate-y-1 mt-1 border-2 border-white">
              <Mail size={26} className="sm:mr-2" /><span className="hidden sm:inline font-black tracking-wide">輸出 PDF 並發送信件</span>
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
