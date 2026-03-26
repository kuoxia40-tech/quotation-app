
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  
  <script src="https://cdn.tailwindcss.com"></script>
  <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

  <style>
    /* 徹底隱藏數字輸入框的上下箭頭 (支援所有瀏覽器) */
    input[type=number]::-webkit-inner-spin-button, 
    input[type=number]::-webkit-outer-spin-button { 
      -webkit-appearance: none !important; 
      margin: 0 !important; 
      display: none !important;
    }
    input[type=number] { -moz-appearance: textfield !important; }
    
    /* 徹底覆寫 iOS Safari 的巨型下拉選單箭頭，改用極小自訂箭頭節省空間 */
    select { 
      -webkit-appearance: none !important;
      -moz-appearance: none !important;
      appearance: none !important;
      background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%23475569' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpolyline points='6 9 12 15 18 9'%3E%3C/polyline%3E%3C/svg%3E") !important;
      background-repeat: no-repeat !important;
      background-position: right 1px center !important;
      background-size: 10px !important;
      padding-right: 12px !important; 
    }
    input[type=date] { background-color: transparent; }

    /* 列印排版最佳化 */
    @media print {
      @page { margin: 10mm; size: A4 portrait; }
      body { margin: 0; background-color: white !important; -webkit-print-color-adjust: exact; print-color-adjust: exact; }
      tr, li { page-break-inside: avoid; break-inside: avoid; }
      .print-container { max-width: 100% !important; width: 100% !important; margin: 0 !important; padding: 0 !important; box-shadow: none !important; border: none !important; }
      /* 列印時隱藏下拉選單的自訂箭頭 */
      select { background-image: none !important; padding-right: 0 !important; }
      input[type=date]::-webkit-calendar-picker-indicator { display: none; }
    }
  </style>
</head>
<body class="bg-slate-100 text-slate-800">

  <div id="root">
    <div class="min-h-screen flex items-center justify-center text-slate-500 font-bold text-xl">
      系統載入中，請稍候...
    </div>
  </div>

  <script type="text/babel">
    const { useState, useEffect } = React;

    // --- 圖示元件 ---
    const Download = ({ size=24, className="" }) => <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="7 10 12 15 17 10"></polyline><line x1="12" y1="15" x2="12" y2="3"></line></svg>;
    const Plus = ({ className="" }) => <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><line x1="12" y1="5" x2="12" y2="19"></line><line x1="5" y1="12" x2="19" y2="12"></line></svg>;
    const Trash2 = ({ className="" }) => <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path><line x1="10" y1="11" x2="10" y2="17"></line><line x1="14" y1="11" x2="14" y2="17"></line></svg>;
    const X = ({ size=24, className="" }) => <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>;
    const FileText = ({ className="" }) => <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path><polyline points="14 2 14 8 20 8"></polyline><line x1="16" y1="13" x2="8" y2="13"></line><line x1="16" y1="17" x2="8" y2="17"></line><polyline points="10 9 9 9 8 9"></polyline></svg>;
    const RotateCcw = ({ className="" }) => <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><polyline points="1 4 1 10 7 10"></polyline><path d="M3.51 15a9 9 0 1 0 2.13-9.36L1 10"></path></svg>;

    // --- 選單預設資料 ---
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
    const UNIT_OPTIONS = ['口', '式', '迴', '公尺', '個', '組', '台', '套', '坪', '天'];
    const PRESET_REMARKS = [
      '付款方式：訂金 30%、進度款 40%、驗收結案 30%。',
      '本工程不含水泥修補、油漆及磁磚復原。',
      '保固期限：水電管線保固 1 年 (非人為損壞)。',
      '若遇牆體結構過硬或特殊現場環境，施工費用另計。',
      '各項設備以實際挑選之型號為準，多退少補。',
      '施工期間產生之廢棄物由我方負責清運。',
    ];
    const CHINESE_NUMBERS = ['一', '二', '三', '四', '五', '六', '七', '八', '九', '十', '十一', '十二'];

    function App() {
      // 日期處理：改為 YYYY-MM-DD 格式，以供 input type="date" 使用
      const today = new Date();
      const defaultDate = `${today.getFullYear()}-${String(today.getMonth() + 1).padStart(2, '0')}-${String(today.getDate()).padStart(2, '0')}`;
      
      const [headerData, setHeaderData] = useState({ ownerName: '木柵路三段張公館', ownerAddress: '', date: defaultDate });
      const [showTax, setShowTax] = useState(false);

      const [items, setItems] = useState([
        { id: 'c1', type: 'category', col1: '燈具及電力工程', isCustomCol1: false },
        { id: '1', type: 'item', col1: '1', col2: '開關出線口', qty: 40, unit: '口', price: 1500, remark: '', isCustomCol2: false, isCustomUnit: false },
        { id: '2', type: 'item', col1: '2', col2: '崁燈出線口', qty: 65, unit: '口', price: 500, remark: '', isCustomCol2: false, isCustomUnit: false },
        { id: '3', type: 'item', col1: '3', col2: '陽台&主燈&變壓器出線口', qty: 15, unit: '口', price: 500, remark: '', isCustomCol2: false, isCustomUnit: false },
        { id: '4', type: 'item', col1: '5', col2: '崁燈&主燈安裝費', qty: 1, unit: '式', price: 22000, remark: '', isCustomCol2: false, isCustomUnit: false },
        { id: 'c2', type: 'category', col1: '專用迴路及插座', isCustomCol1: false },
        { id: '5', type: 'item', col1: '1', col2: '冷氣專用迴路', qty: 6, unit: '迴', price: 3500, remark: '5.5mm²', isCustomCol2: false, isCustomUnit: false },
        { id: '6', type: 'item', col1: '2', col2: '廚房/陽台/冰箱專用迴路', qty: 2, unit: '迴', price: 3500, remark: '5.5mm²', isCustomCol2: false, isCustomUnit: false },
      ]);

      // 底部合約備註狀態
      const [footerRemarks, setFooterRemarks] = useState([
        { id: 'f1', text: PRESET_REMARKS[0], isCustom: false },
        { id: 'f2', text: PRESET_REMARKS[1], isCustom: false },
        { id: 'f3', text: PRESET_REMARKS[2], isCustom: false },
        { id: 'f4', text: PRESET_REMARKS[3], isCustom: false },
      ]);

      const [grandTotal, setGrandTotal] = useState(0);
      const [modal, setModal] = useState({ show: false, message: '', onConfirm: null });

      // 計算總計
      useEffect(() => {
        let total = 0;
        items.forEach(item => {
          if (item.type === 'item') { total += (parseFloat(item.qty) || 0) * (parseFloat(item.price) || 0); }
        });
        setGrandTotal(total);
      }, [items]);

      const handleHeaderChange = (field, value) => setHeaderData(prev => ({ ...prev, [field]: value }));
      const handleItemChange = (id, field, value) => setItems(prev => prev.map(item => item.id === id ? { ...item, [field]: value } : item));
      const deleteItem = (id) => setItems(prev => prev.filter(item => item.id !== id));

      // 在最底部新增一列
      const addItem = () => {
        setItems(prev => {
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
          return [...prev, { id: Date.now().toString(), type: 'item', col1: (lastSuffix + 1).toString(), col2: '', qty: 1, unit: '式', price: 0, remark: '', isCustomCol2: false, isCustomUnit: false }];
        });
      };

      // 在指定的列「正下方」插入新的一列
      const insertItemBelow = (id) => {
        setItems(prev => {
          const index = prev.findIndex(i => i.id === id);
          if (index === -1) return prev;
          
          const currentItem = prev[index];
          let nextSuffix = '';
          if (currentItem.col1 && !isNaN(parseInt(currentItem.col1))) {
            nextSuffix = (parseInt(currentItem.col1) + 1).toString();
          }
          
          const newItem = {
            id: Date.now().toString(), type: 'item', col1: nextSuffix, col2: '', qty: 1, 
            unit: currentItem.unit || '式', price: 0, remark: '', isCustomCol2: false, isCustomUnit: false
          };
          
          const newArray = [...prev];
          newArray.splice(index + 1, 0, newItem);
          return newArray;
        });
      };

      const addCategory = () => setItems(prev => [...prev, { id: Date.now().toString(), type: 'category', col1: CATEGORY_OPTIONS[0], isCustomCol1: false }]);

      // 備註條款的增刪改查
      const updateRemark = (id, newText) => setFooterRemarks(prev => prev.map(r => r.id === id ? { ...r, text: newText } : r));
      const toggleCustomRemark = (id, isCustom) => setFooterRemarks(prev => prev.map(r => r.id === id ? { ...r, isCustom, text: isCustom ? '' : r.text } : r));
      const deleteRemark = (id) => setFooterRemarks(prev => prev.filter(r => r.id !== id));
      const addRemark = () => setFooterRemarks(prev => [...prev, { id: Date.now().toString(), text: PRESET_REMARKS[0], isCustom: false }]);

      const formatNumber = (num) => Number(num).toLocaleString('en-US');

      const handleExportPDF = () => {
        setModal({
          show: true,
          message: '系統即將為您產生精美報價單。\n\n請在接下來的列印視窗中，將印表機目的地選擇為「另存為 PDF」，即可將檔案儲存至您的設備中發送。',
          onConfirm: () => {
            setModal({ show: false, message: '', onConfirm: null });
            setTimeout(() => window.print(), 300);
          }
        });
      };

      // 樣式設定：手機版字型稍微再縮小一點點以完美塞入 100% 螢幕
      const inputClass = "w-full bg-transparent outline-none p-0.5 sm:p-2 hover:bg-white focus:bg-white focus:ring-1 focus:ring-blue-300 transition-colors text-[10px] sm:text-[15px]";
      const tdClass = "border border-slate-400 p-0 sm:p-1";
      const thClass = "border border-slate-400 bg-[#b4c6e7] p-1 sm:p-2 text-center font-bold text-[10px] sm:text-[15px] text-slate-800";
      
      let currentCatIndex = 0;

      return (
        <div className="min-h-screen bg-slate-200 py-2 sm:py-8 px-1 sm:px-8 pb-32 print:p-0 print:bg-white font-sans">
          
          {modal.show && (
            <div className="fixed inset-0 bg-slate-900/60 flex items-center justify-center z-50 print:hidden backdrop-blur-sm">
              <div className="bg-white p-6 rounded-2xl shadow-2xl max-w-md w-full mx-4 border-t-4 border-blue-600">
                <h3 className="text-xl font-bold mb-4 flex items-center text-blue-800">
                  <FileText className="mr-2 text-blue-600 w-6 h-6" />
                  儲存 PDF 檔
                </h3>
                <p className="text-slate-600 whitespace-pre-line leading-relaxed mb-6">{modal.message}</p>
                <div className="flex justify-end gap-3">
                  <button onClick={() => setModal({ show: false, message: '', onConfirm: null })} className="px-4 py-2 text-slate-600 bg-slate-100 rounded-lg hover:bg-slate-200 transition font-medium">取消</button>
                  <button onClick={modal.onConfirm} className="px-4 py-2 text-white bg-blue-600 rounded-lg hover:bg-blue-700 flex items-center transition shadow-md font-medium">
                    <Download className="w-4 h-4 mr-2" />
                    了解並繼續
                  </button>
                </div>
              </div>
            </div>
          )}

          <div className="print-container max-w-5xl mx-auto bg-white shadow-xl rounded-sm sm:rounded-lg p-2 sm:p-12">
            
            {/* 標題 (改成深藍色，稍微縮小手機版字體以防斷行) */}
            <h1 className="text-lg sm:text-3xl font-black text-blue-900 text-center mb-2 sm:mb-6 tracking-widest">
              水電工程維修 / 翻修報價單
            </h1>

            {/* 表頭資訊 */}
            <div className="grid grid-cols-2 gap-x-1 gap-y-1 mb-2 sm:mb-4 text-[10px] sm:text-[15px] border border-slate-400 p-1 sm:p-2 bg-slate-50">
              <div className="flex items-center">
                <span className="font-bold whitespace-nowrap mr-1 w-12 sm:w-20">工程名稱：</span>
                <input type="text" className="w-full bg-transparent outline-none focus:bg-white px-1 font-bold text-blue-900 border-b border-dashed border-slate-300" value={headerData.ownerName} onChange={(e) => handleHeaderChange('ownerName', e.target.value)} placeholder="請輸入名稱" />
              </div>
              <div className="flex items-center">
                <span className="font-bold whitespace-nowrap mr-1 w-12 sm:w-20">公司名稱：</span>
                <span className="w-full px-1">岳鼎水電工程</span>
              </div>
              
              <div className="flex items-center">
                <span className="font-bold whitespace-nowrap mr-1 w-12 sm:w-20">報價日期：</span>
                <input type="date" className="w-full bg-transparent outline-none focus:bg-white px-1 font-bold text-slate-800 border-b border-dashed border-slate-300 cursor-pointer" value={headerData.date} onChange={(e) => handleHeaderChange('date', e.target.value)} />
              </div>
              <div className="flex items-center">
                <span className="font-bold whitespace-nowrap mr-1 w-12 sm:w-20">公司地址：</span>
                <span className="w-full px-1 truncate">新北市蘆洲區光華路150巷13弄1號1樓</span>
              </div>

              <div className="flex items-center">
                <span className="font-bold whitespace-nowrap mr-1 w-12 sm:w-20">業主地址：</span>
                <input type="text" className="w-full bg-transparent outline-none focus:bg-white px-1 border-b border-dashed border-slate-300" value={headerData.ownerAddress} onChange={(e) => handleHeaderChange('ownerAddress', e.target.value)} placeholder="請輸入地址" />
              </div>
              <div className="flex items-center">
                <span className="font-bold whitespace-nowrap mr-1 w-12 sm:w-20">聯絡人：</span>
                <span className="w-full px-1">廖家緯 (0988676742)</span>
              </div>
            </div>

            {/* 表格區塊：重新調配比例寬度，讓備註跟項目名稱有更多空間 */}
            <div className="w-full border-2 border-slate-400 print:border-slate-800">
              <table className="w-full table-fixed border-collapse">
                <thead>
                  <tr>
                    <th className={`${thClass} w-[6%]`}>編號</th>
                    <th className={`${thClass} w-[27%] text-left`}>項目名稱及規格</th>
                    <th className={`${thClass} w-[8%]`}>數量</th>
                    <th className={`${thClass} w-[9%]`}>單位</th>
                    <th className={`${thClass} w-[13%]`}>單價</th>
                    <th className={`${thClass} w-[16%]`}>複價</th>
                    <th className={`${thClass} w-[21%]`}>備註</th>
                  </tr>
                </thead>
                <tbody>
                  {items.map((item) => {
                    if (item.type === 'category') {
                      currentCatIndex++;
                      const prefixText = (CHINESE_NUMBERS[currentCatIndex - 1] || currentCatIndex) + '、';
                      return (
                        <tr key={item.id} className="border-b border-slate-400 bg-slate-100 group">
                          <td colSpan="7" className="p-0 border border-slate-400 relative">
                            <div className="flex items-center px-1">
                              <span className="font-bold text-slate-800 text-[10px] sm:text-[15px] whitespace-nowrap shrink-0">{prefixText}</span>
                              {item.isCustomCol1 ? (
                                <div className="flex items-center w-full">
                                  <input type="text" className={`${inputClass} font-bold text-slate-800 ml-1`} value={item.col1} onChange={(e) => handleItemChange(item.id, 'col1', e.target.value)} placeholder="自訂名稱..." autoFocus />
                                  <button onClick={() => handleItemChange(item.id, 'isCustomCol1', false)} className="p-1 text-slate-400 print:hidden shrink-0"><RotateCcw className="w-3 h-3 sm:w-4 sm:h-4" /></button>
                                </div>
                              ) : (
                                <div className="flex items-center w-full">
                                  <select className="w-full p-0.5 sm:p-2 bg-transparent outline-none font-bold text-slate-800 text-[10px] sm:text-[15px] cursor-pointer hover:bg-slate-200 transition" value={item.col1} onChange={(e) => { if (e.target.value === 'CUSTOM') { setItems(prev => prev.map(i => i.id === item.id ? { ...i, isCustomCol1: true, col1: '' } : i)); } else { handleItemChange(item.id, 'col1', e.target.value); } }}>
                                    <option value="" disabled>請選擇分類...</option>
                                    {CATEGORY_OPTIONS.map(opt => <option key={opt} value={opt}>{opt}</option>)}
                                    <option value="CUSTOM" className="text-blue-600">✎ 自訂輸入...</option>
                                  </select>
                                </div>
                              )}
                            </div>
                            <button onClick={() => deleteItem(item.id)} className="absolute right-1 top-1/2 -translate-y-1/2 text-red-400 hover:text-red-600 p-1 opacity-0 group-hover:opacity-100 transition print:hidden bg-white rounded shadow-sm"><Trash2 className="w-3 h-3 sm:w-4 sm:h-4" /></button>
                          </td>
                        </tr>
                      );
                    }

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
                              <button onClick={() => handleItemChange(item.id, 'isCustomCol2', false)} className="p-0.5 text-slate-400 print:hidden shrink-0"><RotateCcw className="w-3 h-3 sm:w-4 sm:h-4" /></button>
                            </div>
                          ) : (
                            <select className="w-full p-0.5 sm:p-2 bg-transparent outline-none cursor-pointer hover:bg-white truncate text-[10px] sm:text-[15px]" value={item.col2} onChange={(e) => { if (e.target.value === 'CUSTOM') { setItems(prev => prev.map(i => i.id === item.id ? { ...i, isCustomCol2: true, col2: '' } : i)); } else { handleItemChange(item.id, 'col2', e.target.value); } }}>
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
                          {item.isCustomUnit ? (
                            <div className="flex items-center w-full overflow-hidden">
                              <input type="text" className={`${inputClass} truncate text-center`} value={item.unit} onChange={(e) => handleItemChange(item.id, 'unit', e.target.value)} placeholder="自訂" autoFocus />
                              <button onClick={() => handleItemChange(item.id, 'isCustomUnit', false)} className="p-0.5 text-slate-400 print:hidden shrink-0"><RotateCcw className="w-3 h-3 sm:w-4 sm:h-4" /></button>
                            </div>
                          ) : (
                            <select className="w-full p-0.5 sm:p-2 bg-transparent outline-none cursor-pointer hover:bg-white text-center text-[10px] sm:text-[15px]" value={item.unit} onChange={(e) => { if (e.target.value === 'CUSTOM') { setItems(prev => prev.map(i => i.id === item.id ? { ...i, isCustomUnit: true, unit: '' } : i)); } else { handleItemChange(item.id, 'unit', e.target.value); } }}>
                              {UNIT_OPTIONS.map(opt => <option key={opt} value={opt}>{opt}</option>)}
                              {!UNIT_OPTIONS.includes(item.unit) && item.unit !== '' && <option value={item.unit}>{item.unit}</option>}
                              <option value="CUSTOM" className="text-blue-600">✎ 自訂</option>
                            </select>
                          )}
                        </td>
                        <td className={tdClass}>
                          <input type="number" className={`${inputClass} text-right`} value={item.price} onChange={(e) => handleItemChange(item.id, 'price', e.target.value)} min="0" />
                        </td>
                        <td className={tdClass}>
                          <div className="w-full text-right p-0.5 sm:p-2 font-medium text-[10px] sm:text-[15px]">{formatNumber(subtotal)}</div>
                        </td>
                        <td className={tdClass}>
                          <div className="flex items-center justify-between h-full">
                            <input type="text" className={`${inputClass} text-left text-slate-500 min-w-0 flex-grow`} value={item.remark} onChange={(e) => handleItemChange(item.id, 'remark', e.target.value)} placeholder="備註..." />
                            
                            {/* 將操作按鈕縮小，避免在手機上擋住備註文字 */}
                            <div className="flex items-center shrink-0 print:hidden space-x-0.5 opacity-70 hover:opacity-100 transition-opacity">
                               <button onClick={() => insertItemBelow(item.id)} className="text-green-600 hover:bg-green-100 p-0.5 rounded border border-slate-200 bg-slate-50 shadow-sm" title="在下方新增一列"><Plus className="w-2.5 h-2.5 sm:w-3.5 sm:h-3.5" /></button>
                               <button onClick={() => deleteItem(item.id)} className="text-red-500 hover:bg-red-100 p-0.5 rounded border border-slate-200 bg-slate-50 shadow-sm" title="刪除"><Trash2 className="w-2.5 h-2.5 sm:w-3.5 sm:h-3.5" /></button>
                            </div>
                          </div>
                        </td>
                      </tr>
                    );
                  })}
                  
                  {/* 空白列 */}
                  <tr>
                    <td className={tdClass}><div className="h-4 sm:h-6"></div></td>
                    <td className={tdClass}></td><td className={tdClass}></td><td className={tdClass}></td><td className={tdClass}></td><td className={tdClass}></td><td className={tdClass}></td>
                  </tr>

                  {/* 總結算與自訂備註條款區塊 */}
                  <tr>
                    <td colSpan="4" className="border border-slate-400 p-1 sm:p-2 text-[10px] sm:text-[13px] text-slate-700 font-medium align-top" rowSpan={showTax ? 3 : 1}>
                      <ul className="list-disc pl-4 space-y-1">
                        {footerRemarks.map((remark) => (
                          <li key={remark.id} className="relative group/remark">
                            {remark.isCustom ? (
                              <div className="flex items-center w-full">
                                <input className="w-full bg-transparent border-b border-dashed border-slate-300 outline-none focus:bg-white focus:text-blue-700 py-0.5" value={remark.text} onChange={(e) => updateRemark(remark.id, e.target.value)} placeholder="請輸入自訂條款..." autoFocus />
                                <button onClick={() => toggleCustomRemark(remark.id, false)} className="ml-1 print:hidden text-slate-400 shrink-0"><RotateCcw className="w-3 h-3" /></button>
                                <button onClick={() => deleteRemark(remark.id)} className="ml-1 print:hidden text-red-400 shrink-0"><Trash2 className="w-3 h-3" /></button>
                              </div>
                            ) : (
                              <div className="flex items-center w-full">
                                <select className="w-full bg-transparent outline-none cursor-pointer hover:bg-slate-100 truncate py-0.5" value={remark.text} onChange={(e) => { if (e.target.value === 'CUSTOM') toggleCustomRemark(remark.id, true); else updateRemark(remark.id, e.target.value); }}>
                                  <option value="" disabled>請選擇條款...</option>
                                  {PRESET_REMARKS.map(r => <option key={r} value={r}>{r}</option>)}
                                  {!PRESET_REMARKS.includes(remark.text) && remark.text !== '' && <option value={remark.text}>{remark.text}</option>}
                                  <option value="CUSTOM" className="text-blue-600">✎ 自訂條款...</option>
                                </select>
                                <button onClick={() => deleteRemark(remark.id)} className="ml-1 print:hidden text-red-400 shrink-0"><Trash2 className="w-3 h-3" /></button>
                              </div>
                            )}
                          </li>
                        ))}
                      </ul>
                      <button onClick={addRemark} className="mt-1 flex items-center text-blue-500 hover:bg-blue-50 print:hidden text-xs font-bold px-1 py-1 rounded transition w-max"><Plus className="w-3 h-3 mr-1"/>新增合約條款</button>
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
                {showTax ? <X size={14} className="mr-1" /> : <Plus className="w-3.5 h-3.5 mr-1" />}
                {showTax ? '移除 8% 營業稅' : '新增 8% 營業稅'}
              </button>
            </div>
          </div>

          {/* 右下角浮動按鈕 */}
          <div className="fixed bottom-3 right-3 sm:bottom-6 sm:right-6 flex flex-col items-end gap-2 print:hidden z-40">
            <button onClick={addCategory} className="bg-cyan-600 hover:bg-cyan-700 text-white w-10 h-10 sm:w-auto sm:h-auto sm:px-4 sm:py-2.5 rounded-full shadow-lg flex items-center justify-center transition font-bold" title="新增分類">
              <Plus className="w-5 h-5 sm:mr-2" /><span className="hidden sm:inline text-sm">新增分類</span>
            </button>
            <button onClick={addItem} className="bg-sky-500 hover:bg-sky-600 text-white w-10 h-10 sm:w-auto sm:h-auto sm:px-4 sm:py-2.5 rounded-full shadow-lg flex items-center justify-center transition font-bold" title="新增列於底部">
              <Plus className="w-5 h-5 sm:mr-2" /><span className="hidden sm:inline text-sm">新增列於底部</span>
            </button>
            <button onClick={handleExportPDF} className="bg-blue-700 hover:bg-blue-800 text-white w-12 h-12 sm:w-auto sm:h-auto sm:px-5 sm:py-3.5 rounded-full shadow-xl flex items-center justify-center transition transform hover:-translate-y-1 mt-1 border-2 border-white">
              <Download size={22} className="sm:mr-2" /><span className="hidden sm:inline font-black tracking-wide text-sm">儲存 PDF 檔</span>
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
