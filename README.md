<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover" />

  <script src="https://cdn.tailwindcss.com"></script>
  <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

  <style>
    :root {
      color-scheme: light;
    }

    html, body {
      min-height: 100%;
    }

    body {
      -webkit-font-smoothing: antialiased;
      text-rendering: optimizeLegibility;
    }

    input[type=number]::-webkit-inner-spin-button,
    input[type=number]::-webkit-outer-spin-button {
      -webkit-appearance: none;
      margin: 0;
    }

    input[type=number] {
      -moz-appearance: textfield;
    }

    input,
    select,
    textarea,
    button {
      font-size: 16px;
    }

    @media print {
      @page { margin: 0; }
      body { margin: 1.4cm; background: white !important; }
      .print-hidden-force { display: none !important; }
      .print-break-inside-avoid { break-inside: avoid; }
    }
  </style>
</head>
<body class="bg-slate-100">
  <div id="root"></div>

  <script type="text/babel">
    const { useMemo, useState } = React;

    const Mail = ({ size=24, className="" }) => (
      <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}>
        <path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"></path>
        <polyline points="22,6 12,13 2,6"></polyline>
      </svg>
    );

    const Plus = ({ size=24, className="" }) => (
      <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}>
        <line x1="12" y1="5" x2="12" y2="19"></line>
        <line x1="5" y1="12" x2="19" y2="12"></line>
      </svg>
    );

    const Trash2 = ({ size=24, className="" }) => (
      <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}>
        <polyline points="3 6 5 6 21 6"></polyline>
        <path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path>
        <line x1="10" y1="11" x2="10" y2="17"></line>
        <line x1="14" y1="11" x2="14" y2="17"></line>
      </svg>
    );

    const X = ({ size=24, className="" }) => (
      <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}>
        <line x1="18" y1="6" x2="6" y2="18"></line>
        <line x1="6" y1="6" x2="18" y2="18"></line>
      </svg>
    );

    const FileText = ({ className="", size=24 }) => (
      <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}>
        <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
        <polyline points="14 2 14 8 20 8"></polyline>
        <line x1="16" y1="13" x2="8" y2="13"></line>
        <line x1="16" y1="17" x2="8" y2="17"></line>
        <polyline points="10 9 9 9 8 9"></polyline>
      </svg>
    );

    const Send = ({ className="", size=18 }) => (
      <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}>
        <line x1="22" y1="2" x2="11" y2="13"></line>
        <polygon points="22 2 15 22 11 13 2 9 22 2"></polygon>
      </svg>
    );

    const RotateCcw = ({ size=18, className="" }) => (
      <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}>
        <polyline points="1 4 1 10 7 10"></polyline>
        <path d="M3.51 15a9 9 0 1 0 2.13-9.36L1 10"></path>
      </svg>
    );

    const ChevronDown = ({ size=18, className="" }) => (
      <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}>
        <polyline points="6 9 12 15 18 9"></polyline>
      </svg>
    );

    const Printer = ({ size=18, className="" }) => (
      <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}>
        <polyline points="6 9 6 2 18 2 18 9"></polyline>
        <path d="M6 18H4a2 2 0 0 1-2-2v-5a2 2 0 0 1 2-2h16a2 2 0 0 1 2 2v5a2 2 0 0 1-2 2h-2"></path>
        <rect x="6" y="14" width="12" height="8"></rect>
      </svg>
    );

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

    const formatNumber = (num) => Number(num || 0).toLocaleString('en-US');

    function getStructuredItems(items) {
      let currentCategoryIndex = 0;
      let currentItemIndex = 0;

      return items.map((item) => {
        if (item.type === 'category') {
          currentCategoryIndex += 1;
          currentItemIndex = 0;
          return {
            ...item,
            displayCategoryNo: CHINESE_NUMBERS[currentCategoryIndex - 1] || String(currentCategoryIndex),
            displayCode: '',
            categoryIndex: currentCategoryIndex,
          };
        }

        currentItemIndex += 1;
        return {
          ...item,
          displayCode: currentCategoryIndex > 0 ? `${currentCategoryIndex}.${currentItemIndex}` : `${currentItemIndex}`,
          categoryIndex: currentCategoryIndex,
          itemIndex: currentItemIndex,
        };
      });
    }

    function LabeledField({ label, children }) {
      return (
        <div>
          <div className="mb-1 text-xs font-bold tracking-wide text-slate-500">{label}</div>
          {children}
        </div>
      );
    }

    function App() {
      const [headerData, setHeaderData] = useState({
        ownerName: '木柵路三段張公館',
        ownerAddress: '',
      });
      const [showTax, setShowTax] = useState(false);
      const [modal, setModal] = useState({ show: false, mode: 'export' });

      const today = new Date();
      const formattedDate = `${today.getFullYear()}/${String(today.getMonth() + 1).padStart(2, '0')}/${String(today.getDate()).padStart(2, '0')}`;

      const [items, setItems] = useState([
        { id: 'c1', type: 'category', col1: '燈具及電力工程', isCustomCol1: false },
        { id: '1', type: 'item', col2: '開關出線口', qty: 40, unit: '口', price: 1500, remark: '', isCustomCol2: false },
        { id: '2', type: 'item', col2: '崁燈出線口', qty: 65, unit: '口', price: 500, remark: '', isCustomCol2: false },
        { id: '3', type: 'item', col2: '陽台&主燈&變壓器出線口', qty: 15, unit: '口', price: 500, remark: '', isCustomCol2: false },
        { id: '4', type: 'item', col2: '崁燈&主燈安裝費', qty: 1, unit: '式', price: 22000, remark: '', isCustomCol2: false },
        { id: 'c2', type: 'category', col1: '專用迴路及插座', isCustomCol1: false },
        { id: '5', type: 'item', col2: '冷氣專用迴路', qty: 6, unit: '迴', price: 3500, remark: '5.5mm²', isCustomCol2: false },
        { id: '6', type: 'item', col2: '廚房/陽台/冰箱專用迴路', qty: 2, unit: '迴', price: 3500, remark: '5.5mm²', isCustomCol2: false },
      ]);

      const structuredItems = useMemo(() => getStructuredItems(items), [items]);

      const grandTotal = useMemo(() => {
        return structuredItems.reduce((sum, item) => {
          if (item.type !== 'item') return sum;
          return sum + (parseFloat(item.qty) || 0) * (parseFloat(item.price) || 0);
        }, 0);
      }, [structuredItems]);

      const finalTotal = showTax ? Math.round(grandTotal * 1.08) : grandTotal;
      const taxAmount = Math.round(grandTotal * 0.08);

      const mobileGroups = useMemo(() => {
        const groups = [];
        let currentGroup = null;

        structuredItems.forEach((item) => {
          if (item.type === 'category') {
            currentGroup = { category: item, items: [] };
            groups.push(currentGroup);
          } else if (currentGroup) {
            currentGroup.items.push(item);
          } else {
            if (!groups.length || groups[groups.length - 1].category !== null) {
              groups.push({ category: null, items: [] });
            }
            groups[groups.length - 1].items.push(item);
          }
        });

        return groups;
      }, [structuredItems]);

      const handleHeaderChange = (field, value) => {
        setHeaderData((prev) => ({ ...prev, [field]: value }));
      };

      const handleItemChange = (id, field, value) => {
        setItems((prev) => prev.map((item) => item.id === id ? { ...item, [field]: value } : item));
      };

      const deleteItem = (id) => {
        setItems((prev) => prev.filter((item) => item.id !== id));
      };

      const addItem = () => {
        const newId = `item-${Date.now()}`;
        setItems((prev) => ([
          ...prev,
          { id: newId, type: 'item', col2: '', qty: 1, unit: '式', price: 0, remark: '', isCustomCol2: false }
        ]));
      };

      const addCategory = () => {
        const newId = `cat-${Date.now()}`;
        setItems((prev) => ([
          ...prev,
          { id: newId, type: 'category', col1: CATEGORY_OPTIONS[0], isCustomCol1: false }
        ]));
      };

      const openMailDraft = () => {
        const subject = encodeURIComponent(`【報價單】${headerData.ownerName} - 岳鼎水電工程`);
        const body = encodeURIComponent(
`您好，

附件為「${headerData.ownerName}」的水電工程報價單，總金額為 NT$ ${formatNumber(finalTotal)}。

請查閱附件 PDF 檔案，如有任何問題歡迎隨時聯繫。

聯絡人：廖家緯 (0988676742)

謝謝！`
        );
        window.location.href = `mailto:?subject=${subject}&body=${body}`;
      };

      const inputClass = "w-full rounded-xl border border-slate-200 bg-white px-3 py-2.5 text-slate-800 outline-none transition placeholder:text-slate-400 focus:border-blue-400 focus:ring-4 focus:ring-blue-100 print:border-0 print:bg-transparent print:px-0 print:py-0 print:ring-0";
      const compactInputClass = "w-full rounded-lg border border-slate-200 bg-white px-2.5 py-2 text-slate-800 outline-none transition focus:border-blue-400 focus:ring-4 focus:ring-blue-100 print:border-0 print:bg-transparent print:px-0 print:py-0 print:ring-0";

      return (
        <div className="min-h-screen bg-slate-100 px-3 py-4 pb-36 text-slate-800 sm:px-6 sm:py-8 sm:pb-36 print:bg-white print:px-0 print:py-0 print:pb-0">
          {modal.show && (
            <div className="fixed inset-0 z-50 flex items-end justify-center bg-slate-900/55 p-3 backdrop-blur-sm sm:items-center print:hidden">
              <div className="w-full max-w-lg rounded-3xl bg-white p-5 shadow-2xl sm:p-6">
                <div className="mb-4 flex items-center gap-3">
                  <div className="flex h-11 w-11 items-center justify-center rounded-2xl bg-blue-100 text-blue-700">
                    <FileText size={22} />
                  </div>
                  <div>
                    <div className="text-lg font-black text-slate-900">輸出報價單</div>
                    <div className="text-sm text-slate-500">手機與電腦都可分開操作，不再用延遲跳轉。</div>
                  </div>
                </div>

                <div className="space-y-3 text-sm leading-6 text-slate-600">
                  <div className="rounded-2xl bg-slate-50 p-4">
                    1. 先按「列印 / 另存 PDF」產出檔案。<br />
                    2. 再按「開啟 Email 草稿」自動帶入主旨與內文。<br />
                    3. 把剛存好的 PDF 夾帶進郵件即可。
                  </div>
                </div>

                <div className="mt-5 grid grid-cols-1 gap-3 sm:grid-cols-3">
                  <button
                    onClick={() => setModal({ show: false, mode: 'export' })}
                    className="rounded-2xl bg-slate-100 px-4 py-3 font-bold text-slate-600 transition hover:bg-slate-200"
                  >
                    關閉
                  </button>
                  <button
                    onClick={() => {
                      setModal({ show: false, mode: 'export' });
                      window.print();
                    }}
                    className="flex items-center justify-center gap-2 rounded-2xl bg-blue-50 px-4 py-3 font-bold text-blue-700 transition hover:bg-blue-100"
                  >
                    <Printer size={18} />
                    列印 / 另存 PDF
                  </button>
                  <button
                    onClick={() => {
                      setModal({ show: false, mode: 'export' });
                      openMailDraft();
                    }}
                    className="flex items-center justify-center gap-2 rounded-2xl bg-blue-700 px-4 py-3 font-bold text-white transition hover:bg-blue-800"
                  >
                    <Send size={16} />
                    開啟 Email 草稿
                  </button>
                </div>
              </div>
            </div>
          )}

          <div className="mx-auto max-w-6xl rounded-3xl bg-white p-4 shadow-xl sm:p-8 lg:p-10 print:max-w-none print:rounded-none print:p-0 print:shadow-none">
            <div className="mb-5 border-b-4 border-blue-900 pb-4 text-center print:mb-6 print:border-b-2">
              <h1 className="text-2xl font-black tracking-[0.18em] text-blue-900 sm:text-4xl">
                水電工程維修 / 翻修報價單
              </h1>
            </div>

            <div className="mb-6 grid grid-cols-1 overflow-hidden rounded-2xl border-2 border-slate-300 print:rounded-none print:border-slate-800 sm:grid-cols-2">
              <div className="flex border-b border-slate-300 sm:border-r print:border-slate-800">
                <div className="flex w-24 shrink-0 items-center bg-slate-100 px-3 py-3 text-sm font-bold text-slate-700 print:border-r print:border-slate-800">公司名稱</div>
                <div className="flex w-full items-center px-3 py-3 text-sm font-black tracking-wider text-blue-900">岳鼎水電工程</div>
              </div>
              <div className="flex border-b border-slate-300 print:border-slate-800">
                <div className="flex w-24 shrink-0 items-center bg-slate-100 px-3 py-3 text-sm font-bold text-slate-700 print:border-r print:border-slate-800">業主名稱</div>
                <input type="text" className="w-full px-3 py-3 outline-none focus:bg-blue-50 print:bg-transparent" value={headerData.ownerName} onChange={(e) => handleHeaderChange('ownerName', e.target.value)} placeholder="請輸入業主名稱" />
              </div>
              <div className="flex border-b border-slate-300 sm:border-r print:border-slate-800">
                <div className="flex w-24 shrink-0 items-center bg-slate-100 px-3 py-3 text-sm font-bold text-slate-700 print:border-r print:border-slate-800">公司地址</div>
                <div className="flex w-full items-center px-3 py-3 text-sm text-slate-700">新北市蘆洲區光華路150巷13弄1號1樓</div>
              </div>
              <div className="flex border-b border-slate-300 print:border-slate-800">
                <div className="flex w-24 shrink-0 items-center bg-slate-100 px-3 py-3 text-sm font-bold text-slate-700 print:border-r print:border-slate-800">業主地址</div>
                <input type="text" className="w-full px-3 py-3 outline-none focus:bg-blue-50 print:bg-transparent" value={headerData.ownerAddress} onChange={(e) => handleHeaderChange('ownerAddress', e.target.value)} placeholder="請輸入業主地址" />
              </div>
              <div className="flex border-b border-slate-300 sm:border-b-0 sm:border-r print:border-slate-800">
                <div className="flex w-24 shrink-0 items-center bg-slate-100 px-3 py-3 text-sm font-bold text-slate-700 print:border-r print:border-slate-800">聯絡人</div>
                <div className="flex w-full items-center px-3 py-3 text-sm font-bold text-slate-700">廖家緯 (0988676742)</div>
              </div>
              <div className="flex">
                <div className="flex w-24 shrink-0 items-center bg-slate-100 px-3 py-3 text-sm font-bold text-slate-700 print:border-r print:border-slate-800">報價日期</div>
                <div className="flex w-full items-center px-3 py-3 text-sm text-slate-700">{formattedDate}</div>
              </div>
            </div>

            <div className="mb-4 flex items-center justify-between gap-3 rounded-2xl bg-slate-50 p-3 print:hidden sm:p-4">
              <div>
                <div className="text-sm font-bold text-slate-700">報價設定</div>
                <div className="text-xs text-slate-500 sm:text-sm">手機版改成卡片式操作，桌面與列印維持正式表格。</div>
              </div>
              <button
                onClick={() => setShowTax(!showTax)}
                className={`inline-flex items-center gap-2 rounded-2xl px-4 py-3 text-sm font-bold transition ${showTax ? 'bg-red-50 text-red-600 hover:bg-red-100' : 'bg-slate-200 text-slate-700 hover:bg-slate-300'}`}
              >
                {showTax ? <X size={18} /> : <Plus size={18} />}
                {showTax ? '移除 8% 營業稅' : '新增 8% 營業稅'}
              </button>
            </div>

            <div className="block md:hidden print:hidden">
              <div className="space-y-4">
                {mobileGroups.map((group, groupIndex) => (
                  <div key={`group-${groupIndex}`} className="space-y-3">
                    {group.category && (
                      <div className="rounded-2xl border border-blue-200 bg-blue-50 p-4 shadow-sm">
                        <div className="mb-3 flex items-center justify-between gap-3">
                          <div className="text-base font-black text-blue-900">
                            {group.category.displayCategoryNo}、分類
                          </div>
                          <button
                            onClick={() => deleteItem(group.category.id)}
                            className="rounded-xl bg-white p-2 text-red-500 shadow-sm"
                            title="刪除分類"
                          >
                            <Trash2 size={16} />
                          </button>
                        </div>

                        {group.category.isCustomCol1 ? (
                          <div className="flex items-center gap-2">
                            <input
                              type="text"
                              className={inputClass}
                              value={group.category.col1}
                              onChange={(e) => handleItemChange(group.category.id, 'col1', e.target.value)}
                              placeholder="請輸入自訂分類名稱"
                              autoFocus
                            />
                            <button
                              onClick={() => handleItemChange(group.category.id, 'isCustomCol1', false)}
                              className="rounded-xl bg-white p-3 text-slate-500 shadow-sm"
                              title="切回下拉選單"
                            >
                              <RotateCcw />
                            </button>
                          </div>
                        ) : (
                          <div className="relative">
                            <select
                              className={`${inputClass} appearance-none pr-10`}
                              value={group.category.col1}
                              onChange={(e) => {
                                if (e.target.value === 'CUSTOM') {
                                  handleItemChange(group.category.id, 'isCustomCol1', true);
                                  handleItemChange(group.category.id, 'col1', '');
                                } else {
                                  handleItemChange(group.category.id, 'col1', e.target.value);
                                }
                              }}
                            >
                              <option value="" disabled>請選擇分類...</option>
                              {CATEGORY_OPTIONS.map(opt => <option key={opt} value={opt}>{opt}</option>)}
                              <option value="CUSTOM">✎ 自訂輸入...</option>
                            </select>
                            <ChevronDown className="pointer-events-none absolute right-3 top-1/2 -translate-y-1/2 text-slate-400" />
                          </div>
                        )}
                      </div>
                    )}

                    {group.items.map((item) => {
                      const subtotal = (parseFloat(item.qty) || 0) * (parseFloat(item.price) || 0);

                      return (
                        <div key={item.id} className="rounded-2xl border border-slate-200 bg-white p-4 shadow-sm">
                          <div className="mb-4 flex items-center justify-between gap-3">
                            <div>
                              <div className="text-xs font-bold tracking-wide text-slate-500">編號</div>
                              <div className="text-lg font-black text-blue-900">{item.displayCode}</div>
                            </div>
                            <button
                              onClick={() => deleteItem(item.id)}
                              className="rounded-xl bg-red-50 p-2.5 text-red-500"
                              title="刪除項目"
                            >
                              <Trash2 size={16} />
                            </button>
                          </div>

                          <div className="space-y-4">
                            <LabeledField label="項目名稱及規格規範">
                              {item.isCustomCol2 ? (
                                <div className="flex items-center gap-2">
                                  <input
                                    type="text"
                                    className={inputClass}
                                    value={item.col2}
                                    onChange={(e) => handleItemChange(item.id, 'col2', e.target.value)}
                                    placeholder="請輸入自訂項目"
                                    autoFocus
                                  />
                                  <button
                                    onClick={() => handleItemChange(item.id, 'isCustomCol2', false)}
                                    className="rounded-xl bg-slate-100 p-3 text-slate-500"
                                    title="切回下拉選單"
                                  >
                                    <RotateCcw />
                                  </button>
                                </div>
                              ) : (
                                <div className="relative">
                                  <select
                                    className={`${inputClass} appearance-none pr-10`}
                                    value={item.col2}
                                    onChange={(e) => {
                                      if (e.target.value === 'CUSTOM') {
                                        handleItemChange(item.id, 'isCustomCol2', true);
                                        handleItemChange(item.id, 'col2', '');
                                      } else {
                                        handleItemChange(item.id, 'col2', e.target.value);
                                      }
                                    }}
                                  >
                                    <option value="" disabled>請選擇項目...</option>
                                    {ITEM_OPTIONS.map(opt => <option key={opt} value={opt}>{opt}</option>)}
                                    <option value="CUSTOM">✎ 自訂輸入...</option>
                                  </select>
                                  <ChevronDown className="pointer-events-none absolute right-3 top-1/2 -translate-y-1/2 text-slate-400" />
                                </div>
                              )}
                            </LabeledField>

                            <div className="grid grid-cols-2 gap-3">
                              <LabeledField label="數量">
                                <input
                                  type="number"
                                  className={inputClass}
                                  value={item.qty}
                                  onChange={(e) => handleItemChange(item.id, 'qty', e.target.value)}
                                  min="0"
                                />
                              </LabeledField>
                              <LabeledField label="單位">
                                <input
                                  type="text"
                                  className={inputClass}
                                  value={item.unit}
                                  onChange={(e) => handleItemChange(item.id, 'unit', e.target.value)}
                                />
                              </LabeledField>
                            </div>

                            <div className="grid grid-cols-2 gap-3">
                              <LabeledField label="單價">
                                <input
                                  type="number"
                                  className={inputClass}
                                  value={item.price}
                                  onChange={(e) => handleItemChange(item.id, 'price', e.target.value)}
                                  min="0"
                                />
                              </LabeledField>
                              <LabeledField label="複價">
                                <div className="rounded-xl border border-slate-200 bg-slate-50 px-3 py-2.5 text-right font-black text-slate-800">
                                  {formatNumber(subtotal)}
                                </div>
                              </LabeledField>
                            </div>

                            <LabeledField label="備註說明">
                              <input
                                type="text"
                                className={inputClass}
                                value={item.remark}
                                onChange={(e) => handleItemChange(item.id, 'remark', e.target.value)}
                                placeholder="備註..."
                              />
                            </LabeledField>
                          </div>
                        </div>
                      );
                    })}
                  </div>
                ))}
              </div>

              <div className="mt-4 overflow-hidden rounded-2xl border border-slate-200 bg-white shadow-sm">
                <div className="flex items-center justify-between border-b border-slate-100 px-4 py-3">
                  <div className="text-sm font-bold text-slate-600">{showTax ? '報價金額' : '總計金額'}</div>
                  <div className="text-xl font-black text-red-600">{formatNumber(grandTotal)}</div>
                </div>
                {showTax && (
                  <>
                    <div className="flex items-center justify-between border-b border-slate-100 px-4 py-3 text-sm">
                      <div className="font-bold text-slate-600">營業稅 (8%)</div>
                      <div className="font-bold text-slate-800">{formatNumber(taxAmount)}</div>
                    </div>
                    <div className="flex items-center justify-between bg-blue-50 px-4 py-3">
                      <div className="font-black text-blue-900">含稅總金額</div>
                      <div className="text-2xl font-black text-red-600">{formatNumber(finalTotal)}</div>
                    </div>
                  </>
                )}
              </div>
            </div>

            <div className="hidden overflow-x-auto rounded-2xl border-2 border-blue-800 md:block print:block print:rounded-none print:border-slate-800">
              <table className="w-full min-w-[980px] border-collapse print:min-w-0">
                <thead>
                  <tr className="bg-blue-800 text-white print:bg-slate-200 print:text-black">
                    <th className="w-20 border-r border-blue-700 p-3 text-center text-sm font-bold print:border-slate-800">編號</th>
                    <th className="min-w-[260px] border-r border-blue-700 p-3 text-left text-sm font-bold print:border-slate-800">項目名稱及規格規範</th>
                    <th className="w-24 border-r border-blue-700 p-3 text-center text-sm font-bold print:border-slate-800">數量</th>
                    <th className="w-20 border-r border-blue-700 p-3 text-center text-sm font-bold print:border-slate-800">單位</th>
                    <th className="w-28 border-r border-blue-700 p-3 text-right text-sm font-bold print:border-slate-800">單價</th>
                    <th className="w-32 border-r border-blue-700 p-3 text-right text-sm font-bold print:border-slate-800">複價</th>
                    <th className="w-52 p-3 text-left text-sm font-bold">備註說明</th>
                  </tr>
                </thead>
                <tbody>
                  {structuredItems.map((item) => {
                    if (item.type === 'category') {
                      return (
                        <tr key={item.id} className="group border-b border-slate-300 bg-blue-50 print:bg-slate-100">
                          <td colSpan="6" className="border-r border-slate-300 p-2 print:border-slate-800">
                            {item.isCustomCol1 ? (
                              <div className="flex items-center gap-2 px-2">
                                <span className="shrink-0 whitespace-nowrap text-lg font-black text-blue-900">{item.displayCategoryNo}、</span>
                                <input
                                  type="text"
                                  className={`${compactInputClass} text-lg font-black text-blue-900`}
                                  value={item.col1}
                                  onChange={(e) => handleItemChange(item.id, 'col1', e.target.value)}
                                  placeholder="請輸入自訂分類名稱"
                                  autoFocus
                                />
                                <button
                                  onClick={() => handleItemChange(item.id, 'isCustomCol1', false)}
                                  className="rounded-lg p-2 text-slate-400 transition hover:bg-white hover:text-blue-600 print:hidden"
                                  title="切回下拉選單"
                                >
                                  <RotateCcw size={16} />
                                </button>
                              </div>
                            ) : (
                              <div className="relative flex items-center px-2">
                                <span className="shrink-0 whitespace-nowrap text-lg font-black text-blue-900">{item.displayCategoryNo}、</span>
                                <select
                                  className="w-full appearance-none rounded-lg bg-transparent p-2 pr-10 text-lg font-black text-blue-900 outline-none transition hover:bg-blue-100 focus:bg-white focus:ring-4 focus:ring-blue-100 print:appearance-none"
                                  value={item.col1}
                                  onChange={(e) => {
                                    if (e.target.value === 'CUSTOM') {
                                      handleItemChange(item.id, 'isCustomCol1', true);
                                      handleItemChange(item.id, 'col1', '');
                                    } else {
                                      handleItemChange(item.id, 'col1', e.target.value);
                                    }
                                  }}
                                >
                                  <option value="" disabled>請選擇分類...</option>
                                  {CATEGORY_OPTIONS.map(opt => <option key={opt} value={opt}>{opt}</option>)}
                                  <option value="CUSTOM">✎ 自訂輸入...</option>
                                </select>
                                <ChevronDown size={16} className="pointer-events-none absolute right-4 top-1/2 -translate-y-1/2 text-blue-500 print:hidden" />
                              </div>
                            )}
                          </td>
                          <td className="bg-blue-50 p-2 text-center align-middle print:bg-slate-100">
                            <button
                              onClick={() => deleteItem(item.id)}
                              className="rounded-lg bg-white p-2 text-red-400 opacity-0 shadow-sm transition group-hover:opacity-100 hover:text-red-600 print:hidden"
                              title="刪除分類"
                            >
                              <Trash2 size={16} />
                            </button>
                          </td>
                        </tr>
                      );
                    }

                    const subtotal = (parseFloat(item.qty) || 0) * (parseFloat(item.price) || 0);

                    return (
                      <tr key={item.id} className="group border-b border-slate-300 transition hover:bg-slate-50 print-break-inside-avoid">
                        <td className="border-r border-slate-300 p-2 text-center font-bold text-blue-900 print:border-slate-800">{item.displayCode}</td>
                        <td className="border-r border-slate-300 p-2 print:border-slate-800">
                          {item.isCustomCol2 ? (
                            <div className="flex items-center gap-2">
                              <input
                                type="text"
                                className={compactInputClass}
                                value={item.col2}
                                onChange={(e) => handleItemChange(item.id, 'col2', e.target.value)}
                                placeholder="請輸入自訂項目"
                                autoFocus
                              />
                              <button
                                onClick={() => handleItemChange(item.id, 'isCustomCol2', false)}
                                className="rounded-lg p-2 text-slate-400 transition hover:bg-white hover:text-blue-600 print:hidden"
                                title="切回下拉選單"
                              >
                                <RotateCcw size={14} />
                              </button>
                            </div>
                          ) : (
                            <div className="relative">
                              <select
                                className="w-full appearance-none rounded-lg bg-transparent p-2 pr-10 outline-none transition hover:bg-slate-100 focus:bg-white focus:ring-4 focus:ring-blue-100 print:appearance-none"
                                value={item.col2}
                                onChange={(e) => {
                                  if (e.target.value === 'CUSTOM') {
                                    handleItemChange(item.id, 'isCustomCol2', true);
                                    handleItemChange(item.id, 'col2', '');
                                  } else {
                                    handleItemChange(item.id, 'col2', e.target.value);
                                  }
                                }}
                              >
                                <option value="" disabled>請選擇項目...</option>
                                {ITEM_OPTIONS.map(opt => <option key={opt} value={opt}>{opt}</option>)}
                                <option value="CUSTOM">✎ 自訂輸入...</option>
                              </select>
                              <ChevronDown size={14} className="pointer-events-none absolute right-3 top-1/2 -translate-y-1/2 text-slate-400 print:hidden" />
                            </div>
                          )}
                        </td>
                        <td className="border-r border-slate-300 p-2 print:border-slate-800">
                          <input type="number" className={`${compactInputClass} text-center`} value={item.qty} onChange={(e) => handleItemChange(item.id, 'qty', e.target.value)} min="0" />
                        </td>
                        <td className="border-r border-slate-300 p-2 print:border-slate-800">
                          <input type="text" className={`${compactInputClass} text-center`} value={item.unit} onChange={(e) => handleItemChange(item.id, 'unit', e.target.value)} />
                        </td>
                        <td className="border-r border-slate-300 p-2 print:border-slate-800">
                          <input type="number" className={`${compactInputClass} text-right`} value={item.price} onChange={(e) => handleItemChange(item.id, 'price', e.target.value)} min="0" />
                        </td>
                        <td className="border-r border-slate-300 p-3 text-right font-bold text-slate-800 print:border-slate-800">{formatNumber(subtotal)}</td>
                        <td className="p-2">
                          <div className="flex items-center gap-2">
                            <input type="text" className={`${compactInputClass} text-sm`} value={item.remark} onChange={(e) => handleItemChange(item.id, 'remark', e.target.value)} placeholder="備註..." />
                            <button
                              onClick={() => deleteItem(item.id)}
                              className="rounded-lg bg-slate-100 p-2 text-red-400 opacity-0 transition group-hover:opacity-100 hover:text-red-600 print:hidden"
                              title="刪除項目"
                            >
                              <Trash2 size={16} />
                            </button>
                          </div>
                        </td>
                      </tr>
                    );
                  })}

                  <tr className="border-t-4 border-blue-800 bg-slate-50 print:border-slate-800 print:bg-transparent">
                    <td colSpan="5" className="border-r border-slate-300 p-4 text-right text-base font-black tracking-wider text-blue-900 print:border-slate-800">
                      {showTax ? '報價金額：' : '總計金額：'}
                    </td>
                    <td className="border-r border-slate-300 p-4 text-right text-2xl font-black tracking-wider text-red-600 print:border-slate-800">
                      {formatNumber(grandTotal)}
                    </td>
                    <td className="p-4 text-center text-sm font-medium text-slate-500">(新台幣)</td>
                  </tr>

                  {showTax && (
                    <>
                      <tr className="border-t border-slate-200 bg-slate-50 print:border-slate-800 print:bg-transparent">
                        <td colSpan="5" className="border-r border-slate-300 p-3 text-right font-bold text-slate-700 print:border-slate-800">營業稅 (8%)：</td>
                        <td className="border-r border-slate-300 p-3 text-right text-lg font-bold text-slate-700 print:border-slate-800">{formatNumber(taxAmount)}</td>
                        <td className="p-3"></td>
                      </tr>
                      <tr className="border-t-2 border-slate-400 bg-blue-100 print:border-slate-800 print:bg-transparent">
                        <td colSpan="5" className="border-r border-slate-300 p-4 text-right text-lg font-black tracking-wider text-blue-900 print:border-slate-800">含稅總金額：</td>
                        <td className="border-r border-slate-300 p-4 text-right text-3xl font-black text-red-600 print:border-slate-800">{formatNumber(finalTotal)}</td>
                        <td className="p-4 text-center text-sm font-medium text-slate-500">(新台幣)</td>
                      </tr>
                    </>
                  )}
                </tbody>
              </table>
            </div>

            <div className="mt-6 rounded-2xl border-2 border-slate-300 bg-slate-50 p-4 text-sm leading-7 text-slate-700 print:rounded-none print:border-slate-800 print:bg-transparent sm:p-6">
              <ul className="list-disc space-y-2 pl-5 font-medium sm:pl-6">
                <li>付款方式：訂金 <span className="text-blue-700">30%</span>、進度款 <span className="text-blue-700">40%</span>、驗收結案 <span className="text-blue-700">30%</span>。</li>
                <li>本工程不含水泥修補、油漆及磁磚復原。</li>
                <li>保固期限：水電管線保固 <span className="text-blue-700">1 年</span> (非人為損壞)。</li>
                <li>若遇牆體結構過硬或特殊現場環境，施工費用另計。</li>
              </ul>
            </div>
          </div>

          <div className="fixed bottom-0 left-0 right-0 z-40 border-t border-slate-200 bg-white/95 px-3 pb-[calc(env(safe-area-inset-bottom)+12px)] pt-3 shadow-[0_-8px_30px_rgba(15,23,42,0.08)] backdrop-blur md:hidden print:hidden">
            <div className="grid grid-cols-3 gap-2">
              <button onClick={addCategory} className="flex min-h-[52px] flex-col items-center justify-center rounded-2xl bg-cyan-600 px-2 py-2 text-xs font-bold text-white shadow-sm transition hover:bg-cyan-700">
                <Plus size={18} />
                <span className="mt-1">新增分類</span>
              </button>
              <button onClick={addItem} className="flex min-h-[52px] flex-col items-center justify-center rounded-2xl bg-sky-500 px-2 py-2 text-xs font-bold text-white shadow-sm transition hover:bg-sky-600">
                <Plus size={18} />
                <span className="mt-1">新增項目</span>
              </button>
              <button onClick={() => setModal({ show: true, mode: 'export' })} className="flex min-h-[52px] flex-col items-center justify-center rounded-2xl bg-blue-700 px-2 py-2 text-xs font-bold text-white shadow-sm transition hover:bg-blue-800">
                <Mail size={18} />
                <span className="mt-1">輸出 / 寄信</span>
              </button>
            </div>
          </div>

          <div className="fixed bottom-6 right-6 z-40 hidden flex-col items-end gap-3 md:flex print:hidden">
            <button onClick={addCategory} className="inline-flex items-center gap-2 rounded-full bg-cyan-600 px-5 py-3 font-bold text-white shadow-lg transition hover:bg-cyan-700">
              <Plus size={20} />
              新增分類
            </button>
            <button onClick={addItem} className="inline-flex items-center gap-2 rounded-full bg-sky-500 px-5 py-3 font-bold text-white shadow-lg transition hover:bg-sky-600">
              <Plus size={20} />
              新增項目
            </button>
            <button onClick={() => setModal({ show: true, mode: 'export' })} className="inline-flex items-center gap-2 rounded-full border-2 border-white bg-blue-700 px-6 py-4 font-black tracking-wide text-white shadow-xl transition hover:-translate-y-0.5 hover:bg-blue-800">
              <Mail size={22} />
              輸出 PDF / Email
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
