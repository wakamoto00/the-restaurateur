# the-restaurateur
import React, { useState, useCallback, useEffect, useMemo } from 'react';

// --- 視覺圖示元件 (Inline SVG) ---
const SunIcon = ({ glow, size = 40 }) => (
  <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="#FBBF24" strokeWidth="1.5" strokeLinecap="round" strokeLinejoin="round" 
       className={glow ? "drop-shadow-[0_0_15px_rgba(251,191,36,1)] transition-all duration-1000" : "opacity-40 transition-all duration-1000"}>
    <circle cx="12" cy="12" r="5" fill="#FBBF24" />
    <path d="M12 1v2M12 21v2M4.22 4.22l1.42 1.42M18.36 18.36l1.42 1.42M1 12h2M21 12h2M4.22 19.78l1.42-1.42M18.36 5.64l1.42-1.42" />
  </svg>
);

const MoonIcon = ({ glow, size = 40 }) => (
  <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="#818CF8" strokeWidth="1.5" strokeLinecap="round" strokeLinejoin="round" 
       className={glow ? "drop-shadow-[0_0_15px_rgba(129,140,248,1)] transition-all duration-1000" : "opacity-40 transition-all duration-1000"}>
    <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z" fill="#818CF8" />
  </svg>
);

const ScaleDecoration = () => (
  <svg className="absolute inset-0 w-full h-full opacity-10 pointer-events-none" viewBox="0 0 400 400">
    <circle cx="200" cy="200" r="140" stroke="#E2D1C3" fill="none" strokeWidth="0.5" strokeDasharray="4 8" />
    <circle cx="200" cy="200" r="180" stroke="#E2D1C3" fill="none" strokeWidth="0.2" />
  </svg>
);

// --- 50 項完整事件庫 ---
const ALL_EVENTS = [
  { id: 1, title: "自動化重物搬運", desc: "引進機械手臂輔助內場搬運，打破性別體能偏見。", link: "CEDAW 第 5 條", options: [{ label: "投資輔具 (效率大漲)", effect: { p: 18, e: 15 } }, { label: "維持傳統人力 (省購置費)", effect: { p: 5, e: -15 } }] },
  { id: 2, title: "懷孕主管復職", desc: "主動為產後主管提供技能更新與彈性回歸計畫。", link: "CEDAW 第 11 條", options: [{ label: "啟動留任計畫 (保留戰力)", effect: { p: 12, e: 20 } }, { label: "視同新人 (省管理費)", effect: { p: 5, e: -25 } }] },
  { id: 3, title: "情緒勞動津貼", desc: "外場服務具高度專業，給予對等薪資與支持。", link: "CEDAW 第 11 條", options: [{ label: "專業化敘薪 (提升品質)", effect: { p: 20, e: 15 } }, { label: "嚴格控制工資 (控制開支)", effect: { p: 10, e: -12 } }] },
  { id: 4, title: "騷擾防治零容忍", desc: "即使是大客戶，發生騷擾行為也絕不姑息。", link: "職場安全", options: [{ label: "嚴正拒絕客 (建立品牌感)", effect: { p: 10, e: 30 } }, { label: "息事寧人 (確保短期業績)", effect: { p: 35, e: -30 } }] },
  { id: 5, title: "移工晉升制度", desc: "為優秀移工開設領導課程，拔擢為外場經理。", link: "CEDAW 第 11 條", options: [{ label: "培訓升遷 (跨國客源大增)", effect: { p: 28, e: 20 } }, { label: "維持基層事務 (保穩定)", effect: { p: 5, e: -10 } }] },
  { id: 6, title: "全薪生理假", desc: "提供優於法規全薪生理假，吸引頂尖人才。", link: "CEDAW 第 4 條", options: [{ label: "友善給假 (流動率降)", effect: { p: 15, e: 15 } }, { label: "執行扣薪 (化工時)", effect: { p: 10, e: -20 } }] },
  { id: 7, title: "性別中立制服", desc: "打造無性別美學制服，塑造國際精品形象。", link: "CEDAW 第 5 條", options: [{ label: "形象翻新 (吸引精品客)", effect: { p: 22, e: 15 } }, { label: "傳統剪裁 (省設計費)", effect: { p: 5, e: -10 } }] },
  { id: 8, title: "高齡二度就業", desc: "聘僱資深女性主廚，其秘方成為餐廳冠軍菜。", link: "消除歧視", options: [{ label: "錄取研發 (營收暴漲)", effect: { p: 45, e: 15 } }, { label: "年輕化優先 (保節奏)", effect: { p: 10, e: -10 } }] },
  { id: 9, title: "奢華哺乳空間", desc: "將包廂改裝為頂級哺乳室，成為親子客首選。", link: "CEDAW 第 11 條", options: [{ label: "空間改造 (高端客群增)", effect: { p: 30, e: 20 } }, { label: "維持席次 (確保翻桌率)", effect: { p: 15, e: -15 } }] },
  { id: 10, title: "職場尊重講習", desc: "全員強制參與平權講習，建立高端職場氛圍。", link: "職場文化", options: [{ label: "深度治理 (提升效能)", effect: { p: 15, e: 15 } }, { label: "視為形式 (省營業時間)", effect: { p: 5, e: -20 } }] },
  { id: 11, title: "深夜安心接駁", desc: "提供女性及偏遠員工交通補助，穩定產能。", link: "職場安全", options: [{ label: "提供接駁 (穩定夜班)", effect: { p: 20, e: 12 } }, { label: "員工自理 (省預算)", effect: { p: 5, e: -10 } }] },
  { id: 12, title: "同性伴侶福利", desc: "福利全面與法定配偶看齊，吸引多元人才。", link: "CEDAW 第 1 條", options: [{ label: "全面納入 (聲望提升)", effect: { p: 18, e: 20 } }, { label: "僅限法定 (省保費)", effect: { p: 5, e: -15 } }] },
  { id: 13, title: "薪資透明校正", desc: "主動分析薪資結構，補齊性別薪資缺口。", link: "CEDAW 第 11 條", options: [{ label: "主動補齊 (士氣大漲)", effect: { p: 10, e: 25 } }, { label: "逐年調整 (保現流)", effect: { p: 25, e: -10 } }] },
  { id: 14, title: "全能廚具改造", desc: "引進語音提示設備，聘僱具天賦的身障廚師。", link: "CEDAW 第 4 條", options: [{ label: "智慧改造 (良率提升)", effect: { p: 25, e: 18 } }, { label: "維持現況 (省投資)", effect: { p: 10, e: -12 } }] },
  { id: 15, title: "穆斯林友善營運", desc: "通過清真認證並支持員工佩戴特製頭巾。", link: "文化平權", options: [{ label: "認證加開 (擴大客源)", effect: { p: 40, e: 15 } }, { label: "維持原樣 (管理單純)", effect: { p: 10, e: -10 } }] },
  { id: 16, title: "去性別廣告", desc: "以專業實力為廣告重心，吸引注重價值的客群。", link: "CEDAW 第 5 條", options: [{ label: "品牌重塑 (提升溢價)", effect: { p: 25, e: 15 } }, { label: "傳統廣告 (保流量)", effect: { p: 15, e: -10 } }] },
  { id: 17, title: "單親家長彈性班", desc: "針對單親同仁提供特殊排班，穩定核心人力。", link: "友善家庭", options: [{ label: "支持彈性 (降低流失)", effect: { p: 18, e: 15 } }, { label: "一視同仁 (好排班)", effect: { p: 10, e: -8 } }] },
  { id: 18, title: "企業產假專用金", desc: "設立生育基金，優於法律吸引適育頂尖團隊。", link: "CEDAW 第 11 條", options: [{ label: "加碼津貼 (幸福標竿)", effect: { p: 15, e: 20 } }, { label: "法規最低 (控成本)", effect: { p: 20, e: -15 } }] },
  { id: 19, title: "打破職務刻板", desc: "鼓勵女性掌火、男性擺盤，發揮非傳統優勢。", link: "消除偏見", options: [{ label: "混合培訓 (戰力升級)", effect: { p: 20, e: 15 } }, { label: "依習慣分配 (低阻力)", effect: { p: 10, e: -10 } }] },
  { id: 20, title: "全齡成長路徑", desc: "為不同年齡層員工設計客製化成長福利。", link: "人才永續", options: [{ label: "多元路徑 (最穩團隊)", effect: { p: 25, e: 15 } }, { label: "汰弱留強 (降人事費)", effect: { p: 30, e: -20 } }] },
  // ... 其餘 30 題依此類推並包含在高波動 P 值邏輯中
  { id: 21, title: "決策層比例", desc: "強制高階決策會議必須有 40% 女性參與。", link: "CEDAW 第 7 條", options: [{ label: "多元決策 (降錯誤率)", effect: { p: 25, e: 20 } }, { label: "隨緣選拔 (小圈子)", effect: { p: 10, e: -15 } }] },
  { id: 22, title: "多元語言補貼", desc: "獎勵員工用母語接待國際客，擴展市場。", link: "族群平權", options: [{ label: "語言津貼 (外客大漲)", effect: { p: 35, e: 12 } }, { label: "單一語系 (好管理)", effect: { p: 10, e: -5 } }] },
  { id: 23, title: "心理諮商保險", desc: "將心理諮商納入保險，降低隱形倦怠。", link: "職場健康", options: [{ label: "全額保險 (產能提升)", effect: { p: 18, e: 15 } }, { label: "員工自負 (省經費)", effect: { p: 8, e: -5 } }] },
  { id: 24, title: "盲選招募", desc: "履歷去姓名性別，單純實作考選，錄取天才。", link: "消除偏見", options: [{ label: "盲選制度 (創新強)", effect: { p: 30, e: 15 } }, { label: "傳統面試 (省作業)", effect: { p: 10, e: -8 } }] },
  { id: 25, title: "帶薪照顧假", desc: "提供突發家庭照顧支薪假，換取高忠誠度。", link: "CEDAW 第 11 條", options: [{ label: "支持家庭 (留才高)", effect: { p: 15, e: 18 } }, { label: "請假扣薪 (省支出)", effect: { p: 20, e: -20 } }] },
  { id: 26, title: "環境控溫", desc: "改善內場通風，緩解女性生理期不適感。", link: "環境友善", options: [{ label: "環境優化 (效能升)", effect: { p: 28, e: 15 } }, { label: "忍耐是專業 (省電)", effect: { p: 18, e: -10 } }] },
  { id: 27, title: "證照補貼", desc: "全額補貼考取平權或餐飲證照，提升品牌力。", link: "CEDAW 第 10 條", options: [{ label: "證照獎勵 (溢價升)", effect: { p: 25, e: 15 } }, { label: "自行負擔 (省教育金)", effect: { p: 10, e: -5 } }] },
  { id: 28, title: "供應商條款", desc: "優先採購重視 ESG 與平權供應商，獲高合約。", link: "社會責任", options: [{ label: "嚴選鏈結 (獲精品單)", effect: { p: 18, e: 25 } }, { label: "唯利是圖 (採購低)", effect: { p: 40, e: -20 } }] },
  { id: 29, title: "職場教練", desc: "配置專屬教練給女性新人，縮短摸索期成本。", link: "培力計畫", options: [{ label: "教練津貼 (戰力快)", effect: { p: 20, e: 12 } }, { label: "自然適應 (省管理)", effect: { p: 10, e: -5 } }] },
  { id: 30, title: "多元菜單", desc: "開發針對素食、穆斯林的高單價精品套餐。", link: "市場共融", options: [{ label: "研發多元餐 (單價飆)", effect: { p: 50, e: 15 } }, { label: "單一熱銷 (效率高)", effect: { p: 20, e: -5 } }] },
  { id: 31, title: "獨立申訴", desc: "聘請第三方處理騷擾舉報，建立透明信任。", link: "救濟制度", options: [{ label: "外部法務 (留精英)", effect: { p: 15, e: 22 } }, { label: "內部調解 (省律費)", effect: { p: 20, e: -15 } }] },
  { id: 32, title: "遠端混合", desc: "後勤人員開放混合辦公，省辦公租金。", link: "彈性職場", options: [{ label: "混合辦公 (租金省)", effect: { p: 35, e: 15 } }, { label: "全體進駐 (好控時)", effect: { p: 15, e: -10 } }] },
  { id: 33, title: "國際金獎", desc: "參加國際人權企業大賽，獲獎後海外客暴增。", link: "價值傳播", options: [{ label: "報名參賽 (客源漲)", effect: { p: 45, e: 25 } }, { label: "埋頭苦幹 (不名譽)", effect: { p: 15, e: 0 } }] },
  { id: 34, title: "利潤共享", desc: "設計平權 KPI 達標分紅，全員拼品牌價值。", link: "經濟權利", options: [{ label: "利潤分享 (業績拼)", effect: { p: 55, e: 20 } }, { label: "固定薪資 (現流穩)", effect: { p: 25, e: -12 } }] },
  { id: 35, title: "全友善空間", desc: "打造頂級性別友善盥洗室，吸引豪客。", link: "空間平等", options: [{ label: "質感改造 (客價升)", effect: { p: 25, e: 15 } }, { label: "維持二分 (預算留)", effect: { p: 15, e: -5 } }] },
  { id: 36, title: "AI 去標籤", desc: "採購 AI 去標籤甄選系統，避免人為偏見。", link: "消除偏見", options: [{ label: "技術導入 (創新升)", effect: { p: 25, e: 12 } }, { label: "傳統審核 (習慣性)", effect: { p: 15, e: -5 } }] },
  { id: 37, title: "產假保障", desc: "原職原薪回歸並提供獎金，建立業界標竿。", link: "CEDAW 第 11 條", options: [{ label: "100% 保障 (人才穩)", effect: { p: 18, e: 18 } }, { label: "視人力調 (機動性)", effect: { p: 25, e: -15 } }] },
  { id: 38, title: "技術傳承金", desc: "獎勵資深員工傳承給年輕女性，縮短學習。", link: "教育培訓", options: [{ label: "導師制度 (出餐快)", effect: { p: 35, e: 15 } }, { label: "自由發展 (省獎金)", effect: { p: 15, e: -5 } }] },
  { id: 39, title: "彩虹套餐", desc: "推出主題晚宴，獲得新世代消費者極大青睞。", link: "價值行銷", options: [{ label: "主題行銷 (業績破)", effect: { p: 50, e: 20 } }, { label: "傳統促銷 (保險感)", effect: { p: 25, e: -5 } }] },
  { id: 40, title: "家庭補貼", desc: "提供外籍同仁在內的家庭旅遊，向心力最強。", link: "員工福利", options: [{ label: "補助全家 (零離職)", effect: { p: 18, e: 15 } }, { label: "現金獎勵 (現實面)", effect: { p: 30, e: -10 } }] },
  { id: 41, title: "生理精品", desc: "洗手間提供高品質生理用品，回訪率提升。", link: "實質平等", options: [{ label: "免費供應 (口碑轉)", effect: { p: 25, e: 15 } }, { label: "自動販賣 (微利)", effect: { p: 5, e: -5 } }] },
  { id: 42, title: "男產假倡導", desc: "鼓勵男主管休滿產假，翻轉企業加班文化。", link: "打破刻板", options: [{ label: "帶頭示範 (形象佳)", effect: { p: 12, e: 25 } }, { label: "順其自然 (不干涉)", effect: { p: 10, e: -10 } }] },
  { id: 43, title: "女廚特訓", desc: "專為女廚設計控火特訓，內場實力倍增。", link: "CEDAW 第 10 條", options: [{ label: "特訓計畫 (效能升)", effect: { p: 25, e: 18 } }, { label: "混合訓練 (省場租)", effect: { p: 12, e: -5 } }] },
  { id: 44, title: "深夜車隊", desc: "簽約專屬安心車隊，保障員工通勤絕對安全。", link: "職場安全", options: [{ label: "專屬接送 (排班韌)", effect: { p: 22, e: 12 } }, { label: "自行叫車 (省金)", effect: { p: 10, e: -10 } }] },
  { id: 45, title: "平權宣言", desc: "宣布餐廳願景以平權為第一獲利指標。", link: "永續經營", options: [{ label: "願景發布 (獲投資)", effect: { p: 40, e: 30 } }, { label: "保持低調 (保守性)", effect: { p: 25, e: -10 } }] },
  { id: 46, title: "薪資稽核", desc: "聘請四大會計師事務所進行薪資透明稽查。", link: "CEDAW 第 11 條", options: [{ label: "透明稽核 (融資優)", effect: { p: 30, e: 20 } }, { label: "自行估算 (省費用)", effect: { p: 12, e: -5 } }] },
  { id: 47, title: "混合彈性", desc: "後勤全面彈性上下班，避開塞車提高產能。", link: "現代管理", options: [{ label: "彈性管理 (時效升)", effect: { p: 28, e: 12 } }, { label: "全員點名 (控人力)", effect: { p: 18, e: -10 } }] },
  { id: 48, title: "停業培力", desc: "淡季停業一天進行全員深度平權培力。", link: "教育推廣", options: [{ label: "停業訓練 (凝聚力)", effect: { p: 20, e: 25 } }, { label: "發放手冊 (省營業)", effect: { p: 8, e: -5 } }] },
  { id: 49, title: "限定晚宴", desc: "開發「平衡之衡」套餐，資助平權活動。", link: "品牌形象", options: [{ label: "盛大販售 (營業額)", effect: { p: 65, e: 20 } }, { label: "常規菜單 (平穩)", effect: { p: 25, e: 0 } }] },
  { id: 50, title: "接班選拔", desc: "選拔最具平衡智慧接班人，不分背景性別。", link: "永續領導", options: [{ label: "多元接班 (命最長)", effect: { p: 35, e: 25 } }, { label: "嫡系交接 (慣性穩)", effect: { p: 25, e: -15 } }] }
];

// --- 15 項隨機突發事件 ---
const SUDDEN_EVENTS = [
  { title: "【法規大稽查】", desc: "政府實施性平檢討。若你的平權價值不足，將面臨停工重罰。", impact: { p: -35, e: 0 } },
  { title: "【社群媒體盛讚】", desc: "您的友善政策被廣為分享，引發全國性的訂位爆發。", impact: { p: 45, e: 15 } },
  { title: "【國際金獎認證】", desc: "餐廳獲頒平權餐飲最高殊榮，全球加盟金大幅翻倍。", impact: { p: 60, e: 30 } },
  { title: "【惡意抹黑危機】", desc: "競爭對手針對平權理念發動抹黑，需撥預算進行公關戰。", impact: { p: -25, e: 5 } },
  { title: "【集體離職潮】", desc: "因忽視基層員工的心聲，引發核心團隊抗議離職。", impact: { p: -30, e: -20 } },
  { title: "【ESG 報告優選】", desc: "被評為年度 ESG 揭露最透明企業，吸引大型金控入股。", impact: { p: 35, e: 20 } },
  { title: "【供應商醜聞】", desc: "核心供應商被踢爆壓榨勞工，您的品牌連帶遭受抵制。", impact: { p: -20, e: -25 } },
  { title: "【數位轉型成功】", desc: "全新的彈性排班系統研發成功，人力配置成本大幅下降。", impact: { p: 40, e: 10 } },
  { title: "【育兒友善標章】", desc: "榮獲官方頒發標章，政府補助金與租稅優惠本月入帳。", impact: { p: 25, e: 15 } },
  { title: "【薪資缺口爆料】", desc: "匿名員工爆料內部薪資依然存在缺口，名譽受損。", impact: { p: -15, e: -20 } },
  { title: "【員工心理健康月】", desc: "諮商計畫成效顯著，全員離職率創下歷史新低。", impact: { p: 20, e: 10 } },
  { title: "【平權大會主辦】", desc: "主辦業界平權大會，成為領導品牌，溢價能力提升。", impact: { p: 30, e: 20 } },
  { title: "【限定聯名大賣】", desc: "與人權組織聯名推出的公益餐具被搶購一空，營收新高。", impact: { p: 55, e: 15 } },
  { title: "【隱形天花板抗議】", desc: "部分員工反應升遷制度黑箱，團隊向心力受到衝擊。", impact: { p: -10, e: -15 } },
  { title: "【國際巨星造訪】", desc: "知名平權活動家私下造訪並發文推廣，品牌效應驚人。", impact: { p: 40, e: 10 } }
];

// --- 藝術天秤組件 ---
const ArtisticScale = ({ p, e }) => {
  const rotation = useMemo(() => ((p / (p + e)) * 2 - 1) * 25, [p, e]); 
  return (
    <div className="relative w-full flex flex-col items-center py-16">
      <ScaleDecoration />
      <div className="relative w-72 h-40 flex flex-col items-center justify-end">
        <div 
          className="absolute w-full h-[1.5px] bg-gradient-to-r from-amber-500/60 via-[#E2D1C3] to-indigo-500/60 transition-transform duration-1000 ease-in-out"
          style={{ transform: `rotate(${rotation}deg)`, bottom: '50px' }}
        >
          <div className="absolute -left-10 -top-14 flex flex-col items-center">
             <div className="h-14 w-[0.5px] bg-[#E2D1C3]/20" />
             <SunIcon glow={p >= e} size={48} />
          </div>
          <div className="absolute -right-10 -top-14 flex flex-col items-center">
             <div className="h-14 w-[0.5px] bg-[#E2D1C3]/20" />
             <MoonIcon glow={e > p} size={48} />
          </div>
          <div className="absolute left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 w-4 h-4 rounded-full border border-white/20 bg-[#0E101A] shadow-xl" />
        </div>
        <div className="w-[1px] h-28 bg-gradient-to-t from-transparent via-[#E2D1C3]/40 to-[#E2D1C3]/60 mb-[-10px]" />
        <div className="w-12 h-[1px] bg-[#E2D1C3]/30 rounded-full" />
      </div>
      <div className="mt-10 flex gap-24 text-[9px] tracking-[0.5em] font-bold uppercase opacity-30 select-none">
        <span className="text-amber-500">Capital</span>
        <span className="text-indigo-400">Values</span>
      </div>
    </div>
  );
};

export default function App() {
  const [gameState, setGameState] = useState('start'); 
  const [p, setP] = useState(50);
  const [e, setE] = useState(50);
  const [sessionEvents, setSessionEvents] = useState([]);
  const [currentIndex, setCurrentIndex] = useState(0);
  const [currentSudden, setCurrentSudden] = useState(null);

  const startNewGame = () => {
    const selected = [...ALL_EVENTS].sort(() => Math.random() - 0.5).slice(0, 20);
    setSessionEvents(selected);
    setCurrentIndex(0); setP(50); setE(50);
    setGameState('playing');
  };

  const handleDecision = (opt) => {
    const nextP = Math.min(100, Math.max(0, p + opt.effect.p));
    const nextE = Math.min(100, Math.max(0, e + opt.effect.e));
    setP(nextP); setE(nextE);

    if (currentIndex > 0 && Math.random() < 0.28) {
      const s = SUDDEN_EVENTS[Math.floor(Math.random() * SUDDEN_EVENTS.length)];
      setCurrentSudden(s);
      setGameState('sudden');
    } else {
      nextTurn(nextP, nextE);
    }
  };

  const nextTurn = (cP, cE) => {
    if (currentIndex >= sessionEvents.length - 1 || cP <= 0 || cE <= 0) {
      setGameState('result');
    } else {
      setCurrentIndex(prev => prev + 1);
      setGameState('playing');
    }
  };

  const proceedSudden = () => {
    const nextP = Math.min(100, Math.max(0, p + (currentSudden.impact.p || 0)));
    const nextE = Math.min(100, Math.max(0, e + (currentSudden.impact.e || 0)));
    setP(nextP); setE(nextE);
    setCurrentSudden(null);
    nextTurn(nextP, nextE);
  };

  const getEvaluation = () => {
    const diff = Math.abs(p - e);
    if (diff <= 5) return { t: "傳奇主理人", d: "您達成了一種近乎不可能的『藝術級平衡』。獲利不再是冷冰冰的數字，而是伴隨著尊嚴而生的能量。", c: "text-amber-500" };
    if (diff <= 18) return { t: "卓越經營者", d: "您在驚濤駭浪的市場中守住了品牌的核心，成功在利潤與價值間維持了優雅的動態軌跡。", c: "text-[#E2D1C3]" };
    return { t: "失衡的願景", d: "傾斜的天秤預示了未來。缺乏平權支持的利潤僅是海市蜃樓，請重新審視您的經營本質。", c: "text-rose-500" };
  };

  return (
    <div className="min-h-screen bg-[#0E101A] text-slate-300 font-sans flex flex-col items-center justify-center p-6 select-none overflow-hidden relative">
      <div className="absolute inset-0 pointer-events-none opacity-[0.04]" style={{ backgroundImage: `url("data:image/svg+xml,%3Csvg viewBox='0 0 250 250' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.75' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E")` }}></div>

      <div className="text-center mb-12 animate-in fade-in duration-1000 z-10">
        <span className="text-[10px] tracking-[1em] text-white/30 uppercase font-bold">Standalone Experience</span>
        <h1 className="text-4xl font-serif italic text-[#E2D1C3] mt-3 tracking-widest shadow-inner">The Restaurateur</h1>
      </div>

      <div className="w-full max-w-sm bg-white/[0.02] border border-white/10 rounded-[4rem] overflow-hidden flex flex-col min-h-[660px] shadow-[0_50px_100px_-20px_rgba(0,0,0,0.8)] backdrop-blur-2xl relative z-10 transition-all">
        
        <div className="bg-white/[0.01] border-b border-white/5">
          <ArtisticScale p={p} e={e} />
        </div>

        <div className="flex-grow p-10 flex flex-col relative">
          
          {gameState === 'start' && (
            <div className="flex flex-col items-center justify-center text-center h-full space-y-14 animate-in zoom-in duration-1000">
              <p className="text-[#E2D1C3]/70 italic font-serif leading-relaxed text-2xl px-2">
                「在高效經營的日光下，<br/>守望共融平衡的月輝。」
              </p>
              <button 
                onClick={startNewGame}
                className="group relative px-12 py-5 rounded-full border border-amber-500/40 hover:border-amber-400 transition-all active:scale-95 shadow-[0_0_30px_rgba(251,191,36,0.1)]"
              >
                <span className="relative font-bold tracking-[0.5em] text-amber-500 uppercase text-xs">啟動經營模擬</span>
              </button>
            </div>
          )}

          {gameState === 'playing' && sessionEvents[currentIndex] && (
            <div className="flex flex-col h-full animate-in fade-in slide-in-from-right-12 duration-700">
              <div className="flex justify-between items-center mb-8">
                <span className="text-[9px] font-black text-amber-500/80 bg-amber-500/10 px-4 py-1.5 rounded-full border border-amber-500/20 uppercase tracking-widest">Case {currentIndex + 1} / 20</span>
                <span className="text-[8px] text-white/30 uppercase tracking-[0.2em] font-bold">{sessionEvents[currentIndex].link}</span>
              </div>
              <h3 className="text-2xl font-serif text-[#E2D1C3] mb-6 leading-tight">{sessionEvents[currentIndex].title}</h3>
              <p className="text-slate-400 text-base leading-relaxed mb-12 italic border-l-2 border-white/5 pl-6">{sessionEvents[currentIndex].desc}</p>
              
              <div className="mt-auto space-y-5">
                {sessionEvents[currentIndex].options.map((opt, i) => (
                  <button 
                    key={i}
                    onClick={() => handleDecision(opt)}
                    className="w-full p-6 text-left rounded-[2rem] border border-white/10 bg-white/[0.03] hover:bg-white/[0.06] hover:border-amber-500/30 transition-all active:bg-amber-500/5 group relative overflow-hidden"
                  >
                    <div className="flex justify-between items-center relative z-10">
                      <span className="text-xs font-bold tracking-widest text-slate-200 uppercase group-hover:text-amber-400 transition-colors">{opt.label}</span>
                      <div className="w-1.5 h-1.5 rounded-full bg-white/10 group-hover:bg-amber-500 shadow-[0_0_10px_#fbbf24] transition-all" />
                    </div>
                  </button>
                ))}
              </div>
            </div>
          )}

          {gameState === 'sudden' && (
            <div className="flex flex-col items-center justify-center h-full text-center animate-in zoom-in-95 duration-500">
              <div className="w-16 h-16 rounded-full border border-rose-500/40 flex items-center justify-center mb-10 text-rose-500">
                <span className="text-3xl font-serif">!</span>
              </div>
              <h3 className="text-rose-500 font-bold tracking-[0.5em] text-xs mb-5 uppercase">{currentSudden.title}</h3>
              <p className="text-slate-300 text-base mb-14 italic leading-relaxed px-4">{currentSudden.desc}</p>
              <button 
                onClick={proceedSudden}
                className="w-full py-5 bg-rose-500/10 border border-rose-500/30 rounded-full text-rose-500 text-[11px] font-bold tracking-[0.4em] uppercase shadow-[0_0_20px_rgba(244,63,94,0.1)] hover:bg-rose-500/20"
              >
                面對挑戰
              </button>
            </div>
          )}

          {gameState === 'result' && (
            <div className="flex flex-col items-center justify-center h-full text-center py-4 animate-in fade-in slide-in-from-bottom-12 duration-1000">
              <span className="text-[10px] tracking-[0.8em] text-white/20 uppercase font-bold mb-8">Strategic Analysis</span>
              <h2 className={`text-4xl font-serif italic mb-10 ${getEvaluation().c}`}>
                {getEvaluation().t}
              </h2>
              <div className="bg-white/[0.02] p-10 rounded-[3rem] border border-white/5 mb-14 w-full italic text-sm text-slate-400 leading-relaxed shadow-inner">
                "{getEvaluation().d}"
              </div>
              <button 
                onClick={() => setGameState('start')}
                className="text-[11px] font-bold text-white/30 uppercase tracking-[0.5em] hover:text-[#E2D1C3] transition-colors"
              >
                Start New Cycle
              </button>
            </div>
          )}
        </div>
      </div>

      <div className="mt-16 opacity-20 text-[9px] tracking-[1em] uppercase text-white font-bold text-center">
        Integrity • Leadership • Vision
      </div>
    </div>
  );
}
