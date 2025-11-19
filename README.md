<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>和平精英地铁军械库 Pro | Metro Weapon Lab</title>
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        :root {
            --bg-dark: #121214;
            --bg-panel: #1e1e24;
            --bg-hover: #2c2c35;
            --accent: #00e5ff;
            --accent-dim: rgba(0, 229, 255, 0.1);
            --compare: #ff4757;
            --compare-dim: rgba(255, 71, 87, 0.1);
            --text-main: #e1e1e6;
            --text-muted: #a8a8b3;
            --success: #00b894;
        }

        body {
            margin: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-dark);
            color: var(--text-main);
            height: 100vh;
            overflow: hidden;
        }

        #app { display: flex; height: 100%; }

        /* 滚动条样式 */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: var(--bg-dark); }
        ::-webkit-scrollbar-thumb { background: #444; border-radius: 3px; }

        /* 左侧列表 */
        .sidebar {
            width: 320px;
            background-color: var(--bg-panel);
            border-right: 1px solid #333;
            display: flex;
            flex-direction: column;
        }

        .brand {
            padding: 20px;
            font-size: 22px;
            font-weight: 900;
            color: var(--accent);
            border-bottom: 1px solid #333;
            letter-spacing: 1px;
        }

        .search-box {
            padding: 15px;
            border-bottom: 1px solid #333;
        }
        .search-input {
            width: 100%;
            background: #121214;
            border: 1px solid #333;
            padding: 10px;
            color: white;
            border-radius: 4px;
            box-sizing: border-box;
        }

        .weapon-list {
            flex: 1;
            overflow-y: auto;
            padding: 10px;
        }

        .weapon-item {
            padding: 12px 15px;
            margin-bottom: 8px;
            background: var(--bg-hover);
            border-radius: 6px;
            cursor: pointer;
            transition: all 0.2s;
            border-left: 3px solid transparent;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .weapon-item:hover { background: #383842; }
        
        .weapon-item.active {
            background: var(--accent-dim);
            border-left-color: var(--accent);
        }

        .weapon-item.comparing {
            background: var(--compare-dim);
            border-left-color: var(--compare);
        }

        .weapon-info h4 { margin: 0; font-size: 14px; }
        .weapon-info span { font-size: 11px; color: var(--text-muted); }

        .compare-btn {
            font-size: 11px;
            padding: 4px 8px;
            border-radius: 4px;
            background: #222;
            border: 1px solid #444;
            color: #888;
            cursor: pointer;
        }
        .compare-btn.is-comparing {
            background: var(--compare);
            color: white;
            border-color: var(--compare);
        }

        /* 右侧主视图 */
        .main-view {
            flex: 1;
            padding: 30px;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 20px;
        }

        .header {
            display: flex;
            justify-content: space-between;
            align-items: flex-end;
            border-bottom: 1px solid #333;
            padding-bottom: 20px;
        }

        .title-group h1 { margin: 0; font-size: 42px; text-transform: uppercase; }
        .title-group .subtitle { color: var(--accent); font-size: 16px; letter-spacing: 2px; font-weight: bold; }
        
        .compare-badge {
            background: linear-gradient(45deg, var(--compare), #ff7675);
            padding: 5px 15px;
            border-radius: 20px;
            font-size: 14px;
            font-weight: bold;
            display: flex;
            align-items: center;
            gap: 8px;
        }

        /* 布局网格 */
        .grid-container {
            display: grid;
            grid-template-columns: 350px 1fr;
            gap: 20px;
        }

        .card {
            background: var(--bg-panel);
            border-radius: 12px;
            padding: 20px;
            border: 1px solid #333;
            position: relative;
        }

        .card-title {
            font-size: 14px;
            color: var(--text-muted);
            text-transform: uppercase;
            margin-bottom: 15px;
            display: flex;
            justify-content: space-between;
        }

        /* 雷达图容器 */
        .radar-container { height: 300px; width: 100%; }

        /* 弹道模拟器 */
        .recoil-box {
            grid-column: 2;
            grid-row: 1 / 3;
            display: flex;
            flex-direction: column;
        }
        
        .canvas-wrapper {
            flex: 1;
            background: radial-gradient(circle, #2a2a2a 1px, transparent 1px) 0 0 / 40px 40px;
            background-color: #000;
            border-radius: 8px;
            position: relative;
            border: 2px solid #333;
            overflow: hidden;
            min-height: 400px;
        }

        .distance-control {
            margin-top: 15px;
            background: #25252b;
            padding: 15px;
            border-radius: 8px;
            display: flex;
            align-items: center;
            gap: 15px;
        }

        input[type=range] { flex: 1; accent-color: var(--accent); }

        /* 对比表格 */
        .compare-table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 10px;
        }
        .compare-table td {
            padding: 12px 5px;
            border-bottom: 1px solid #2c2c35;
            font-size: 14px;
        }
        .compare-table .label { color: var(--text-muted); width: 120px; }
        .val-a { color: var(--accent); font-weight: bold; text-align: right; width: 80px; }
        .val-b { color: var(--compare); font-weight: bold; text-align: right; width: 80px; }
        .diff { font-size: 10px; width: 50px; text-align: center; }
        .diff.better { color: var(--success); }
        .diff.worse { color: var(--compare); }

        .tag {
            display: inline-block;
            padding: 2px 6px;
            background: #333;
            border-radius: 3px;
            font-size: 10px;
            margin-right: 5px;
            color: #aaa;
        }
    </style>
</head>
<body>

<div id="app">
    <div class="sidebar">
        <div class="brand"><i class="fa-solid fa-crosshairs"></i> METRO ARMORY</div>
        <div class="search-box">
            <input type="text" v-model="searchQuery" placeholder="搜索武器 (如: M4, MG3)..." class="search-input">
        </div>
        <div class="weapon-list">
            <div v-for="w in filteredWeapons" :key="w.id" 
                 class="weapon-item" 
                 :class="{ active: currentWeapon.id === w.id, comparing: compareWeapon && compareWeapon.id === w.id }"
                 @click="selectWeapon(w)">
                <div class="weapon-info">
                    <h4>{{ w.name }}</h4>
                    <span>{{ w.type }} | {{ w.ammo }}mm</span>
                </div>
                <button class="compare-btn" 
                        :class="{ 'is-comparing': compareWeapon && compareWeapon.id === w.id }"
                        @click.stop="toggleCompare(w)">
                    {{ compareWeapon && compareWeapon.id === w.id ? 'VS 取消' : 'VS 对比' }}
                </button>
            </div>
        </div>
    </div>

    <div class="main-view">
        <div class="header">
            <div class="title-group">
                <div class="subtitle">{{ currentWeapon.type }}</div>
                <h1>{{ currentWeapon.name }}</h1>
                <div style="display: flex; gap: 10px; margin-top: 5px;">
                    <span class="tag">6级甲单发: {{ currentWeapon.dmg_lv6 }}</span>
                    <span class="tag">射速: {{ currentWeapon.rate }} RPM</span>
                    <span class="tag">弹速: {{ currentWeapon.bulletSpeed }} m/s</span>
                </div>
            </div>
            <div v-if="compareWeapon" class="compare-badge">
                <span>VS</span> {{ compareWeapon.name }}
            </div>
        </div>

        <div class="grid-container">
            
            <div style="display:flex; flex-direction:column; gap:20px;">
                <div class="card">
                    <div class="card-title">性能雷达 (Attributes)</div>
                    <div class="radar-container">
                        <canvas id="radarChart"></canvas>
                    </div>
                </div>

                <div class="card">
                    <div class="card-title">
                        <span>核心数据对比</span>
                        <span style="font-size:10px; color:gray">BASE VS COMPARE</span>
                    </div>
                    <table class="compare-table">
                        <tr v-for="(label, key) in statsMap" :key="key">
                            <td class="label">{{ label }}</td>
                            <td class="val-a">{{ currentWeapon[key] }}</td>
                            <td class="val-b">
                                <span v-if="compareWeapon">{{ compareWeapon[key] }}</span>
                                <span v-else style="color:#333">-</span>
                            </td>
                            <td class="diff" :class="getDiffClass(key)">
                                <span v-if="compareWeapon">{{ getDiffVal(key) }}</span>
                            </td>
                        </tr>
                    </table>
                </div>
            </div>

            <div class="card recoil-box">
                <div class="card-title">
                    <span>弹道散布测试 (Recoil Pattern)</span>
                    <span style="color: var(--accent)">距离: {{ distance }}m</span>
                </div>
                
                <div class="canvas-wrapper" id="canvasContainer">
                    <div style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 10px; height: 10px; background: red; border-radius: 50%; box-shadow: 0 0 10px red; opacity: 0.5;"></div>
                    <div style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 200px; height: 200px; border: 1px solid rgba(255,255,255,0.1); border-radius: 50%;"></div>
                    <canvas id="recoilCanvas" style="width: 100%; height: 100%;"></canvas>
                </div>

                <div class="distance-control">
                    <i class="fa-solid fa-ruler-horizontal" style="color:#666"></i>
                    <span style="font-size: 12px; color:#888; width: 50px;">10m</span>
                    <input type="range" v-model.number="distance" min="10" max="200" step="10">
                    <span style="font-size: 12px; color:#888; width: 50px; text-align: right;">200m</span>
                </div>
                <div style="font-size: 12px; color: #666; margin-top: 10px; text-align: center;">
                    * 弹道基于30发全自动模拟，距离越远，视野内散布被放大 (模拟不压枪情况)
                </div>
            </div>

        </div>
    </div>
</div>

<script>
const { createApp, ref, computed, onMounted, watch, nextTick } = Vue;

createApp({
    setup() {
        // -------------------------------
        // 1. 武器数据库 (20+ 把)
        // -------------------------------
        const weaponDB = [
            // 突击步枪
            { id: 'ar1', name: 'M416 (精制)', type: 'AR', ammo: 5.56, damage: 41, rate: 780, bulletSpeed: 880, stability: 85, mobility: 70, dmg_lv6: 19, cost: 180, patternType: 'stable_up' },
            { id: 'ar2', name: 'AKM (精制)', type: 'AR', ammo: 7.62, damage: 48, rate: 600, bulletSpeed: 715, stability: 55, mobility: 60, dmg_lv6: 24, cost: 150, patternType: 'hard_kick' },
            { id: 'ar3', name: 'M762 (精制)', type: 'AR', ammo: 7.62, damage: 46, rate: 690, bulletSpeed: 715, stability: 50, mobility: 62, dmg_lv6: 22, cost: 160, patternType: 'vertical_zigzag' },
            { id: 'ar4', name: 'SCAR-L', type: 'AR', ammo: 5.56, damage: 41, rate: 690, bulletSpeed: 870, stability: 80, mobility: 72, dmg_lv6: 19, cost: 120, patternType: 'stable_up' },
            { id: 'ar5', name: 'AUG', type: 'AR', ammo: 5.56, damage: 41, rate: 700, bulletSpeed: 940, stability: 90, mobility: 65, dmg_lv6: 19, cost: 200, patternType: 'laser' },
            { id: 'ar6', name: 'Groza', type: 'AR', ammo: 7.62, damage: 47, rate: 750, bulletSpeed: 715, stability: 55, mobility: 65, dmg_lv6: 23, cost: 250, patternType: 'hard_kick' },
            { id: 'ar7', name: 'FAMAS', type: 'AR', ammo: 5.56, damage: 38, rate: 900, bulletSpeed: 920, stability: 60, mobility: 75, dmg_lv6: 17, cost: 190, patternType: 'fast_jitter' },
            { id: 'ar8', name: 'Honey Badger', type: 'AR', ammo: 7.62, damage: 43, rate: 730, bulletSpeed: 680, stability: 65, mobility: 80, dmg_lv6: 20, cost: 170, patternType: 'vertical_zigzag' },
            
            // 冲锋枪
            { id: 'smg1', name: 'P90 (钢铁)', type: 'SMG', ammo: 5.7, damage: 35, rate: 1000, bulletSpeed: 450, stability: 95, mobility: 90, dmg_lv6: 13, cost: 220, patternType: 'cluster' },
            { id: 'smg2', name: 'Vector', type: 'SMG', ammo: 9, damage: 31, rate: 1100, bulletSpeed: 350, stability: 80, mobility: 85, dmg_lv6: 10, cost: 90, patternType: 'fast_rise' },
            { id: 'smg3', name: 'UZI', type: 'SMG', ammo: 9, damage: 26, rate: 1200, bulletSpeed: 350, stability: 85, mobility: 95, dmg_lv6: 8, cost: 60, patternType: 'cluster' },
            { id: 'smg4', name: 'Thompson', type: 'SMG', ammo: .45, damage: 40, rate: 700, bulletSpeed: 280, stability: 60, mobility: 70, dmg_lv6: 15, cost: 70, patternType: 'messy' },
            
            // 射手步枪 & 狙击
            { id: 'dmr1', name: 'MK14 (独眼蛇)', type: 'DMR', ammo: 7.62, damage: 61, rate: 650, bulletSpeed: 853, stability: 25, mobility: 55, dmg_lv6: 34, cost: 300, patternType: 'extreme_kick' },
            { id: 'dmr2', name: 'SKS', type: 'DMR', ammo: 7.62, damage: 53, rate: 400, bulletSpeed: 800, stability: 45, mobility: 60, dmg_lv6: 28, cost: 140, patternType: 'jumpy' },
            { id: 'dmr3', name: 'Mini14', type: 'DMR', ammo: 5.56, damage: 46, rate: 450, bulletSpeed: 990, stability: 75, mobility: 70, dmg_lv6: 22, cost: 130, patternType: 'stable_tap' },
            { id: 'sr1', name: 'AWM (精制)', type: 'SR', ammo: .300, damage: 105, rate: 30, bulletSpeed: 945, stability: 90, mobility: 40, dmg_lv6: 90, cost: 400, patternType: 'single' },
            { id: 'sr2', name: 'M24', type: 'SR', ammo: 7.62, damage: 75, rate: 35, bulletSpeed: 790, stability: 85, mobility: 45, dmg_lv6: 55, cost: 180, patternType: 'single' },
            
            // 轻机枪 & 霰弹
            { id: 'lmg1', name: 'MG3', type: 'LMG', ammo: 7.62, damage: 40, rate: 990, bulletSpeed: 820, stability: 70, mobility: 40, dmg_lv6: 18, cost: 350, patternType: 'linear_rise' },
            { id: 'lmg2', name: 'DP-28', type: 'LMG', ammo: 7.62, damage: 51, rate: 550, bulletSpeed: 715, stability: 90, mobility: 40, dmg_lv6: 25, cost: 100, patternType: 'stable_up' },
            { id: 'sg1', name: 'DBS', type: 'SG', ammo: 12, damage: 240, rate: 100, bulletSpeed: 300, stability: 40, mobility: 60, dmg_lv6: 150, cost: 150, patternType: 'shotgun' }
        ];

        // 状态变量
        const searchQuery = ref('');
        const currentWeapon = ref(weaponDB[0]);
        const compareWeapon = ref(null);
        const distance = ref(50);
        
        // 用于表格对比的键值映射
        const statsMap = {
            damage: '基础伤害',
            dmg_lv6: '6级甲伤害',
            rate: '射速 (RPM)',
            bulletSpeed: '弹速 (m/s)',
            stability: '稳定性评分',
            cost: '造价 (万)'
        };

        // -------------------------------
        // 2. 预生成固定弹道 (Pattern Generation)
        // -------------------------------
        // 为每把枪生成一个固定的 basePattern 数组 [{x, y}, ...] 
        // 这样不管距离怎么变，图案形状本身不变，只是缩放
        const generateFixedPatterns = () => {
            weaponDB.forEach(w => {
                const pts = [];
                let cx = 0, cy = 0; // 每一发的累积偏移量
                
                // 根据枪械类型设定每一发的“后坐力倾向”
                let vKick = 0; // 垂直上跳
                let hKick = 0; // 水平随机
                
                switch(w.patternType) {
                    case 'stable_up': vKick = 2; hKick = 0.5; break; // M4
                    case 'hard_kick': vKick = 5; hKick = 3.0; break; // AKM
                    case 'laser': vKick = 1.5; hKick = 0.2; break; // AUG
                    case 'fast_jitter': vKick = 2.5; hKick = 2.0; break; // FAMAS
                    case 'cluster': vKick = 1.0; hKick = 0.8; break; // P90
                    case 'extreme_kick': vKick = 8.0; hKick = 5.0; break; // MK14
                    case 'single': vKick = 0; hKick = 0; break; // SR
                    case 'shotgun': vKick = 5; hKick = 5; break; // SG
                    default: vKick = 3; hKick = 1;
                }

                // 生成30发弹孔坐标 (相对于原点)
                for(let i=0; i<30; i++) {
                    if (w.type === 'SR' && i > 0) break; // 栓狙只有1发
                    
                    // 1. 固定趋势
                    cy -= vKick * (1 + i * 0.05); // 越往后跳得越高
                    
                    // 2. 左右摆动 (模拟S型弹道)
                    let swing = Math.sin(i * 0.5) * hKick; 
                    // 加上一些随机性但保持整体趋势
                    let randomH = (Math.random() - 0.5) * hKick;
                    cx += swing + randomH;

                    pts.push({ x: cx, y: cy });
                }
                w.basePattern = pts;
            });
        };
        generateFixedPatterns();

        // -------------------------------
        // 3. 逻辑处理
        // -------------------------------
        const filteredWeapons = computed(() => {
            if(!searchQuery.value) return weaponDB;
            return weaponDB.filter(w => w.name.toLowerCase().includes(searchQuery.value.toLowerCase()));
        });

        const selectWeapon = (w) => {
            currentWeapon.value = w;
            if(compareWeapon.value && compareWeapon.value.id === w.id) compareWeapon.value = null;
            refreshViz();
        };

        const toggleCompare = (w) => {
            if (currentWeapon.value.id === w.id) return;
            compareWeapon.value = (compareWeapon.value && compareWeapon.value.id === w.id) ? null : w;
            refreshViz();
        };

        // 对比数值辅助函数
        const getDiffVal = (key) => {
            if(!compareWeapon.value) return '';
            const v1 = currentWeapon.value[key];
            const v2 = compareWeapon.value[key];
            const diff = v2 - v1;
            return diff > 0 ? `+${diff}` : `${diff}`;
        };

        const getDiffClass = (key) => {
            if(!compareWeapon.value) return '';
            const v1 = currentWeapon.value[key];
            const v2 = compareWeapon.value[key];
            
            // 对于后坐力/造价，越低越好？不，这里简化逻辑：数值越大越绿(除造价外?)
            // 假设所有属性越大越好
            if (v2 > v1) return 'better'; // 对比武器更好
            if (v2 < v1) return 'worse';
            return '';
        };

        // -------------------------------
        // 4. 绘图核心 (Canvas & Chart.js)
        // -------------------------------
        let chartInstance = null;

        const drawRadar = () => {
            const ctx = document.getElementById('radarChart');
            if(!ctx) return;
            
            const getVal = (w) => [
                w.damage, w.rate/10, w.bulletSpeed/10, w.stability, w.mobility
            ];

            const data = {
                labels: ['单发威力', '射速', '弹道初速', '操控稳定性', '机动性'],
                datasets: [{
                    label: currentWeapon.value.name,
                    data: getVal(currentWeapon.value),
                    borderColor: '#00e5ff',
                    backgroundColor: 'rgba(0, 229, 255, 0.2)',
                    borderWidth: 2,
                    pointRadius: 0
                }]
            };

            if(compareWeapon.value) {
                data.datasets.push({
                    label: compareWeapon.value.name,
                    data: getVal(compareWeapon.value),
                    borderColor: '#ff4757',
                    backgroundColor: 'rgba(255, 71, 87, 0.1)',
                    borderWidth: 2,
                    pointRadius: 0
                });
            }

            if(chartInstance) {
                chartInstance.data = data;
                chartInstance.update();
            } else {
                chartInstance = new Chart(ctx, {
                    type: 'radar',
                    data: data,
                    options: {
                        scales: {
                            r: {
                                angleLines: { color: '#333' },
                                grid: { color: '#333' },
                                pointLabels: { color: '#e1e1e6', font: {size: 12} },
                                ticks: { display: false, maxTicksLimit: 5 }, // 隐藏刻度数值
                                suggestedMin: 0,
                                suggestedMax: 110
                            }
                        },
                        plugins: { legend: { display: false } },
                        maintainAspectRatio: false
                    }
                });
            }
        };

        const drawRecoil = () => {
            const canvas = document.getElementById('recoilCanvas');
            const container = document.getElementById('canvasContainer');
            if(!canvas || !container) return;

            // 调整分辨率适配 Retina
            canvas.width = container.clientWidth * 2;
            canvas.height = container.clientHeight * 2;
            const ctx = canvas.getContext('2d');
            ctx.scale(2, 2); // 恢复坐标系
            
            const w = container.clientWidth;
            const h = container.clientHeight;
            const centerX = w / 2;
            const centerY = h / 1.5; // 起始点稍微靠下

            ctx.clearRect(0, 0, w, h);

            // 绘制函数
            const plot = (weapon, color, alpha) => {
                if(!weapon.basePattern) return;

                ctx.fillStyle = color;
                ctx.strokeStyle = color;
                ctx.lineWidth = 2;
                
                // 核心逻辑：距离缩放系数
                // 距离越远(distance大)，视野中的弹道看起来越大？
                // 实际上：
                // 游戏里对着墙打，距离越远，墙上的弹孔分布越散 -> Scale 变大
                const scale = distance.value / 15; 

                ctx.beginPath();
                
                weapon.basePattern.forEach((pt, i) => {
                    // 计算缩放后的坐标
                    const px = centerX + (pt.x * scale);
                    const py = centerY + (pt.y * scale);

                    // 模拟距离带来的微小随机抖动 (Bloom)，随距离指数增加
                    // 即使是固定弹道，距离远了也会有散布误差
                    const bloomScale = (distance.value / 50) * (100 - weapon.stability) / 50;
                    const randX = (Math.random() - 0.5) * bloomScale * 5;
                    const randY = (Math.random() - 0.5) * bloomScale * 5;

                    const finalX = px + randX;
                    const finalY = py + randY;

                    // 绘制弹孔
                    ctx.globalAlpha = alpha;
                    ctx.fillRect(finalX - 3, finalY - 3, 6, 6); 

                    // 连线
                    if (i === 0) ctx.moveTo(finalX, finalY);
                    else ctx.lineTo(finalX, finalY);
                });
                ctx.stroke();
            };

            // 先画对比武器 (作为背景，淡一点)
            if(compareWeapon.value) {
                plot(compareWeapon.value, '#ff4757', 0.6);
            }
            // 再画主武器 (高亮)
            plot(currentWeapon.value, '#00e5ff', 1.0);
        };

        const refreshViz = () => {
            nextTick(() => {
                drawRadar();
                drawRecoil();
            });
        };

        watch(distance, drawRecoil);

        onMounted(() => {
            refreshViz();
            window.addEventListener('resize', refreshViz);
        });

        return {
            weaponDB,
            searchQuery,
            filteredWeapons,
            currentWeapon,
            compareWeapon,
            distance,
            selectWeapon,
            toggleCompare,
            statsMap,
            getDiffVal,
            getDiffClass
        };
    }
}).mount('#app');
</script>
</body>
</html>
