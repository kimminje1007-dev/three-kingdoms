[Uploading code_artifact.html…]()
<!DOCTYPE html>
<html lang="ko" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>한반도 삼국통일 시뮬레이터</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Serif+KR:wght@600;700;900&family=Pretendard:wght@400;500;600;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
    <style>
        body {
            font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, sans-serif;
            background-color: #030508;
            color: #e5e7eb;
            margin: 0;
            padding: 0;
            min-height: 100dvh;
            width: 100vw;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        /* 스마트폰 세로 비율(9:16) 모바일 친화적 세로 스크롤 앱 쉘 */
        #app-shell {
            width: 100%;
            min-height: 100dvh;
            max-width: 100%;
            background-color: #070a0f;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            position: relative;
            overflow-y: auto;
            overflow-x: hidden;
            word-break: break-word;
        }
        @media (min-width: 520px) {
            #app-shell {
                min-height: min(96dvh, 920px);
                max-height: 96dvh;
                width: auto;
                aspect-ratio: 9 / 16;
                border-radius: 28px;
                border: 2px solid rgba(255, 255, 255, 0.14);
                box-shadow: 0 24px 70px rgba(0, 0, 0, 0.9), 0 0 0 6px #0b111c;
                overflow-y: auto;
            }
        }
        .serif-kr {
            font-family: 'Noto Serif KR', serif;
        }
        .pixel-canvas {
            image-rendering: pixelated;
            image-rendering: crisp-edges;
        }
        .stat-bar-fill {
            transition: width 0.35s cubic-bezier(0.22, 1, 0.36, 1);
        }
        /* 내륙 60분획 폴리곤 (점선 경계) */
        .prov-cell {
            transition: fill 0.3s ease, fill-opacity 0.25s ease, filter 0.2s ease;
            cursor: pointer;
            stroke: rgba(226, 232, 240, 0.38);
            stroke-width: 0.85px;
            stroke-dasharray: 2.5, 2.5;
            vector-effect: non-scaling-stroke;
        }
        .prov-cell:hover, .prov-cell:active {
            filter: brightness(1.38);
            stroke: #ffffff;
            stroke-dasharray: none;
            stroke-width: 1.4px;
        }
        /* 현대적 정밀 해안선 및 섬 외곽선 (실선) */
        .coast-outline {
            fill: none;
            stroke: #e2e8f0;
            stroke-width: 1.4px;
            stroke-linejoin: round;
            stroke-linecap: round;
            pointer-events: none;
            filter: drop-shadow(0 0 3px rgba(0, 0, 0, 0.9));
        }
        .coast-shelf {
            fill: none;
            stroke: rgba(56, 189, 248, 0.2);
            stroke-width: 4px;
            stroke-linejoin: round;
            stroke-linecap: round;
            pointer-events: none;
        }
        .river-line {
            fill: none;
            stroke: #38bdf8;
            stroke-width: 1.1px;
            stroke-opacity: 0.6;
            stroke-linecap: round;
            stroke-linejoin: round;
            pointer-events: none;
        }
        .sea-route {
            fill: none;
            stroke: rgba(251, 191, 36, 0.3);
            stroke-width: 1px;
            stroke-dasharray: 3, 3;
            pointer-events: none;
        }
        @keyframes pulseGain {
            0%, 100% { filter: brightness(1.25); stroke: #34d399; stroke-width: 2px; stroke-dasharray: none; }
            50% { filter: brightness(1.8) drop-shadow(0 0 8px #10b981); stroke: #ffffff; stroke-width: 2.5px; stroke-dasharray: none; }
        }
        @keyframes pulseLose {
            0%, 100% { filter: brightness(1.15); stroke: #fb7185; stroke-width: 2px; stroke-dasharray: none; }
            50% { filter: brightness(1.7) drop-shadow(0 0 8px #f43f5e); stroke: #fecdd3; stroke-width: 2.5px; stroke-dasharray: none; }
        }
        .prov-gain { animation: pulseGain 0.85s infinite; }
        .prov-lose { animation: pulseLose 0.85s infinite; }

        @keyframes modalPop {
            0% { opacity: 0; transform: scale(0.94) translateY(8px); }
            100% { opacity: 1; transform: scale(1) translateY(0); }
        }
        .modal-animate {
            animation: modalPop 0.22s cubic-bezier(0.16, 1, 0.3, 1) forwards;
        }
        @keyframes titleFloat {
            0%, 100% { transform: translateY(0px); }
            50% { transform: translateY(-4px); }
        }
        .title-float {
            animation: titleFloat 3.5s ease-in-out infinite;
        }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
    </style>
</head>
<body class="select-none">

    <!-- 스마트폰 9:16 세로형 컨테이너 시작 -->
    <div id="app-shell" class="no-scrollbar">

        <!-- 0. 첫 화면: 10종 도트 캐릭터 일러스트 2배 HD 배경 + 타이틀 오버레이 + 방문자 카운터 -->
        <div id="intro-title-screen" class="absolute inset-0 z-40 bg-[#050811] flex flex-col items-center justify-between p-4 overflow-y-auto">
            <!-- 2배 고화질(HD) 도트 캐릭터 일러스트 캔버스 (640x360) -->
            <canvas id="intro-pixel-canvas" width="640" height="360" class="absolute inset-0 w-full h-full object-cover pixel-canvas"></canvas>

            <!-- 비네팅 그라데이션 -->
            <div class="absolute inset-0 bg-gradient-to-b from-black/85 via-black/50 to-black/95 pointer-events-none"></div>

            <!-- 첫 화면 상단 바 -->
            <div class="relative z-10 w-full flex items-center justify-between gap-2 pt-1">
                <div class="inline-flex items-center gap-1.5 px-2.5 py-1 rounded-full bg-amber-500/15 border border-amber-400/40 text-amber-300 text-[10px] font-extrabold truncate max-w-[72%]">
                    <i class="fa-solid fa-gamepad shrink-0"></i>
                    <span id="intro-scene-badge" class="truncate">2배 HD 도트 일러스트 · 고구려 철기병</span>
                    <button onclick="pickRandomIntroScene()" class="ml-1 px-1.5 py-0.5 rounded bg-white/10 hover:bg-white/20 text-[10px] text-white transition shrink-0" title="다른 일러스트 보기">
                        <i class="fa-solid fa-shuffle"></i>
                    </button>
                </div>

                <div class="flex items-center rounded-lg bg-black/70 border border-white/20 p-0.5 text-[10px] font-extrabold shrink-0">
                    <button onclick="setLanguage('ko')" id="btn-intro-lang-ko" class="px-2 py-0.5 rounded bg-amber-500 text-black transition">KOR</button>
                    <button onclick="setLanguage('en')" id="btn-intro-lang-en" class="px-2 py-0.5 rounded text-gray-400 hover:text-white transition">EN</button>
                </div>
            </div>

            <!-- 타이틀 중앙 콘텐츠 박스 -->
            <div class="relative z-10 w-full my-auto p-5 rounded-3xl bg-[#090e17]/90 backdrop-blur-md border border-amber-500/35 shadow-[0_0_40px_rgba(0,0,0,0.9)] text-center flex flex-col items-center my-6">
                <span class="px-2.5 py-0.5 rounded-full bg-amber-400/10 border border-amber-400/30 text-amber-300 text-[10px] font-bold tracking-widest uppercase mb-2">
                    60-REGION EAST ASIA GRAND STRATEGY
                </span>

                <h1 id="intro-main-title" class="serif-kr title-float text-2xl sm:text-3xl font-black text-transparent bg-clip-text bg-gradient-to-r from-amber-200 via-amber-400 to-orange-400 drop-shadow-[0_4px_12px_rgba(0,0,0,0.9)] tracking-tight leading-snug mb-2">
                    한반도 삼국통일 시뮬레이터
                </h1>

                <p id="intro-sub-desc" class="text-xs text-gray-200 leading-relaxed mb-4 font-medium break-words">
                    고구려 · 백제 · 신라의 군주가 되어 60개 구역으로 세분화된 한반도, 만주, 중원, 왜 열도를 제패하는 1턴=10년 역사 시뮬레이션
                </p>

                <div class="grid grid-cols-3 gap-2 w-full mb-5 text-[11px]">
                    <div class="p-2 rounded-xl bg-red-950/60 border border-red-500/40 flex flex-col items-center">
                        <span class="w-2.5 h-2.5 rounded-full bg-red-500 mb-1"></span>
                        <b id="intro-k1-name" class="text-red-300 font-bold">고구려</b>
                        <span id="intro-k1-sub" class="text-[9px] text-gray-300 mt-0.5">철기병 · 대륙정벌</span>
                    </div>
                    <div class="p-2 rounded-xl bg-amber-950/60 border border-amber-500/40 flex flex-col items-center">
                        <span class="w-2.5 h-2.5 rounded-full bg-amber-400 mb-1"></span>
                        <b id="intro-k2-name" class="text-amber-300 font-bold">백제</b>
                        <span id="intro-k2-sub" class="text-[9px] text-gray-300 mt-0.5">대선단 · 해상교역</span>
                    </div>
                    <div class="p-2 rounded-xl bg-emerald-950/60 border border-emerald-500/40 flex flex-col items-center">
                        <span class="w-2.5 h-2.5 rounded-full bg-emerald-400 mb-1"></span>
                        <b id="intro-k3-name" class="text-emerald-300 font-bold">신라</b>
                        <span id="intro-k3-sub" class="text-[9px] text-gray-300 mt-0.5">화랑도 · 삼한일통</span>
                    </div>
                </div>

                <button onclick="startGameFromIntro()" id="btn-intro-start" class="w-full py-3.5 rounded-2xl bg-gradient-to-r from-amber-500 via-amber-400 to-orange-500 hover:from-amber-400 hover:to-orange-400 active:scale-95 text-gray-950 font-black text-base tracking-wide shadow-[0_0_25px_rgba(245,158,11,0.5)] transition flex items-center justify-center gap-2">
                    <i class="fa-solid fa-crown"></i>
                    <span id="lbl-intro-start">시작하기</span>
                </button>

                <div class="mt-3.5 w-full grid grid-cols-2 gap-2 pt-3 border-t border-white/10 text-[11px]">
                    <div class="px-2.5 py-1.5 rounded-xl bg-black/55 border border-white/10 flex items-center justify-between">
                        <span class="text-gray-400 flex items-center gap-1 font-semibold">
                            <i class="fa-regular fa-sun text-amber-400 text-[10px]"></i>
                            <span id="lbl-visit-today">일일 방문자</span>
                        </span>
                        <b id="val-visit-today" class="text-amber-300 font-extrabold tabular-nums">142</b>
                    </div>
                    <div class="px-2.5 py-1.5 rounded-xl bg-black/55 border border-white/10 flex items-center justify-between">
                        <span class="text-gray-400 flex items-center gap-1 font-semibold">
                            <i class="fa-solid fa-users text-emerald-400 text-[10px]"></i>
                            <span id="lbl-visit-total">누적 방문자</span>
                        </span>
                        <b id="val-visit-total" class="text-emerald-300 font-extrabold tabular-nums">3,891</b>
                    </div>
                </div>
            </div>

            <div id="intro-footer-note" class="relative z-10 text-[10px] text-gray-400 font-medium pb-1">
                9:16 모바일 세로 화면 최적화 · 세로 스크롤 및 자동 줄바꿈 지원
            </div>
        </div>

        <!-- 1. 상단 모바일 컴팩트 헤더 -->
        <header class="border-b border-white/10 bg-[#0b1018]/95 backdrop-blur px-2.5 py-1.5 flex items-center justify-between gap-1.5 shrink-0 z-20">
            <div class="flex items-center gap-1.5 min-w-0">
                <span id="hdr-kingdom" class="px-1.5 py-0.5 rounded text-[10px] font-extrabold border shrink-0">고구려</span>
                <h1 id="hdr-ruler" class="serif-kr font-bold text-xs text-white truncate">고담덕 태왕</h1>
                <span class="text-gray-600 text-[10px]">|</span>
                <button onclick="cycleTargetTurns()" id="btn-turn-goal" class="text-[10px] font-bold text-amber-400 hover:underline shrink-0 flex items-center gap-0.5" title="클릭하여 40/60/80턴 변경">
                    <span id="hdr-year">391년 (1/60턴)</span>
                </button>
            </div>

            <div class="flex items-center gap-1 shrink-0">
                <button onclick="initNewRandomGame()" class="px-2 py-1 rounded-md bg-amber-500/15 hover:bg-amber-500/25 border border-amber-500/40 text-amber-300 text-[10px] font-bold transition flex items-center gap-1">
                    <i class="fa-solid fa-dice"></i>
                    <span id="lbl-new-game">새 운명</span>
                </button>
                <div class="flex items-center rounded-md bg-black/60 border border-white/20 p-0.5 text-[10px] font-extrabold">
                    <button onclick="setLanguage('ko')" id="btn-lang-ko" class="px-1.5 py-0.5 rounded bg-amber-500 text-black transition">KOR</button>
                    <button onclick="setLanguage('en')" id="btn-lang-en" class="px-1.5 py-0.5 rounded text-gray-400 hover:text-white transition">EN</button>
                </div>
            </div>
        </header>

        <!-- 2. 모바일 세로형 스크롤 레이아웃 (overflow-y: auto) -->
        <main class="flex-1 flex flex-col p-2 gap-2 overflow-y-auto no-scrollbar">

            <!-- [상단 1] 실시간 국가 상태 2배 HD 도트 배너 + 4대 국력 게이지 -->
            <section class="rounded-xl bg-[#0d131d] border border-white/10 overflow-hidden shrink-0 shadow-md">
                <div class="relative w-full h-16 bg-black">
                    <!-- 2배 고화질(HD) 실시간 상태 배너 캔버스 (680x112) -->
                    <canvas id="dot-banner" width="680" height="112" class="w-full h-full pixel-canvas block"></canvas>
                    <div class="absolute top-1 left-2 right-2 flex items-center justify-between pointer-events-none">
                        <span id="banner-mood" class="px-1.5 py-0.5 rounded bg-black/75 border border-white/15 text-[9px] font-bold text-emerald-300">
                            도성 안정 · 1턴=10년 주기 진행
                        </span>
                        <span class="px-1.5 py-0.5 rounded bg-black/75 border border-white/15 text-[9px] font-bold text-amber-300">
                            <span id="banner-terr-mobile">영토 10 / 60</span>
                            <b id="hdr-terr-count" class="hidden">10</b>
                        </span>
                    </div>
                    <div class="absolute bottom-0 inset-x-0 bg-gradient-to-t from-black/95 to-transparent px-2 py-0.5">
                        <p id="banner-desc" class="text-[9px] text-gray-200 truncate">
                            성벽 위 군사들이 사방을 경계하며 10년의 세월이 흐릅니다.
                        </p>
                    </div>
                </div>

                <div class="grid grid-cols-4 gap-1.5 px-2.5 py-1.5 bg-[#0b0f17] border-t border-white/10 text-[10px]">
                    <div>
                        <div class="flex justify-between leading-none mb-1">
                            <span id="lbl-stat-eco" class="text-gray-400">국고</span>
                            <b id="val-economy" class="text-amber-400">60</b>
                        </div>
                        <div class="h-1 bg-gray-800 rounded-full overflow-hidden">
                            <div id="bar-economy" class="stat-bar-fill h-full bg-amber-400" style="width:60%"></div>
                        </div>
                    </div>
                    <div>
                        <div class="flex justify-between leading-none mb-1">
                            <span id="lbl-stat-mil" class="text-gray-400">군사</span>
                            <b id="val-military" class="text-rose-400">70</b>
                        </div>
                        <div class="h-1 bg-gray-800 rounded-full overflow-hidden">
                            <div id="bar-military" class="stat-bar-fill h-full bg-rose-500" style="width:70%"></div>
                        </div>
                    </div>
                    <div>
                        <div class="flex justify-between leading-none mb-1">
                            <span id="lbl-stat-aut" class="text-gray-400">민심</span>
                            <b id="val-authority" class="text-emerald-400">60</b>
                        </div>
                        <div class="h-1 bg-gray-800 rounded-full overflow-hidden">
                            <div id="bar-authority" class="stat-bar-fill h-full bg-emerald-500" style="width:60%"></div>
                        </div>
                    </div>
                    <div>
                        <div class="flex justify-between leading-none mb-1">
                            <span id="lbl-stat-dip" class="text-gray-400">외교</span>
                            <b id="val-diplomacy" class="text-sky-400">55</b>
                        </div>
                        <div class="h-1 bg-gray-800 rounded-full overflow-hidden">
                            <div id="bar-diplomacy" class="stat-bar-fill h-full bg-sky-400" style="width:55%"></div>
                        </div>
                    </div>
                </div>
            </section>

            <!-- [중앙 2] 실제 해안선 기반 60분획 정밀 동아시아 지도 (선택지 상단 배치) -->
            <section class="h-[210px] rounded-xl bg-[#09121e] border border-white/10 p-1.5 flex flex-col justify-between relative shadow-lg shrink-0">
                <div class="flex items-center justify-between gap-1 px-1 pb-1 border-b border-white/10 text-[9px] shrink-0">
                    <div class="flex items-center gap-1.5 flex-wrap">
                        <span class="flex items-center gap-0.5"><i class="w-2 h-2 rounded-full bg-red-600 inline-block"></i><span id="lgd-goguryeo">고구려</span> <b id="cnt-goguryeo">10</b></span>
                        <span class="flex items-center gap-0.5"><i class="w-2 h-2 rounded-full bg-amber-500 inline-block"></i><span id="lgd-baekje">백제</span> <b id="cnt-baekje">8</b></span>
                        <span class="flex items-center gap-0.5"><i class="w-2 h-2 rounded-full bg-emerald-600 inline-block"></i><span id="lgd-silla">신라</span> <b id="cnt-silla">6</b></span>
                        <span class="flex items-center gap-0.5"><i class="w-2 h-2 rounded-full bg-purple-700 inline-block"></i><span id="lgd-china">중원</span> <b id="cnt-china">18</b></span>
                        <span class="flex items-center gap-0.5"><i class="w-2 h-2 rounded-full bg-slate-600 inline-block"></i><span id="lgd-nomad">초원</span> <b id="cnt-nomad">8</b></span>
                        <span class="flex items-center gap-0.5"><i class="w-2 h-2 rounded-full bg-cyan-600 inline-block"></i><span id="lgd-wa">왜/류큐</span> <b id="cnt-wa">10</b></span>
                    </div>
                    <span id="map-hover-info" class="text-[9px] text-amber-300 font-semibold truncate max-w-[40%]">
                        60분획 정밀 지도
                    </span>
                </div>

                <div class="relative flex-1 w-full h-full flex items-center justify-center bg-[#050c16] rounded-lg overflow-hidden">
                    <svg id="east-asia-map" viewBox="0 0 800 560" preserveAspectRatio="xMidYMid meet" class="w-full h-full block">
                        <defs>
                            <clipPath id="land-clip">
                                <path id="mainland-accurate" d="
                                    M 0,0 L 800,0 L 800,32
                                    L 775,46 L 752,62 L 736,84 L 718,98 L 704,116 L 688,136 L 672,152 L 652,166 L 635,184
                                    L 615,198 L 598,214 L 582,226 L 566,238 L 552,248 L 542,258 L 534,270 L 531,282
                                    L 540,294 L 550,308 L 558,322 L 562,336 L 565,348 L 558,354 L 556,366 L 548,374
                                    L 536,377 L 524,380 L 514,378 L 504,383 L 494,382 L 484,386 L 476,381 L 478,371
                                    L 472,364 L 477,356 L 475,348 L 466,342 L 465,335 L 475,332 L 480,326 L 472,320
                                    L 464,312 L 474,304 L 477,294 L 465,284 L 452,276 L 442,270 L 436,256 L 424,252
                                    L 408,264 L 392,275 L 382,270 L 392,252 L 402,238 L 395,224 L 382,212 L 368,218
                                    L 355,226 L 342,222 L 330,228 L 324,242 L 328,255 L 342,264 L 358,266 L 374,264
                                    L 392,268 L 405,274 L 392,286 L 376,294 L 358,304 L 342,312 L 332,324 L 338,340
                                    L 348,354 L 362,366 L 374,376 L 360,384 L 352,392 L 364,398 L 368,412 L 360,428
                                    L 350,444 L 336,460 L 322,474 L 304,488 L 286,502 L 265,512 L 248,524 L 242,536
                                    L 228,532 L 208,538 L 185,546 L 160,555 L 0,555 Z" />
                            </clipPath>
                            <pattern id="latlon-grid" width="40" height="40" patternUnits="userSpaceOnUse">
                                <path d="M 40 0 L 0 0 0 40" fill="none" stroke="rgba(148, 163, 184, 0.05)" stroke-width="0.7" stroke-dasharray="2,2"/>
                            </pattern>
                        </defs>

                        <rect width="800" height="560" fill="#050d18"/>
                        <rect width="800" height="560" fill="url(#latlon-grid)"/>

                        <g clip-path="url(#land-clip)">
                            <g id="provinces-layer"></g>
                        </g>

                        <g class="coast-outline">
                            <path d="M 775,46 L 752,62 L 736,84 L 718,98 L 704,116 L 688,136 L 672,152 L 652,166 L 635,184 L 615,198 L 598,214 L 582,226 L 566,238 L 552,248 L 542,258 L 534,270 L 531,282 L 540,294 L 550,308 L 558,322 L 562,336 L 565,348 L 558,354 L 556,366 L 548,374 L 536,377 L 524,380 L 514,378 L 504,383 L 494,382 L 484,386 L 476,381 L 478,371 L 472,364 L 477,356 L 475,348 L 466,342 L 465,335 L 475,332 L 480,326 L 472,320 L 464,312 L 474,304 L 477,294 L 465,284 L 452,276 L 442,270 L 436,256 L 424,252 L 408,264 L 392,275 L 382,270 L 392,252 L 402,238 L 395,224 L 382,212 L 368,218 L 355,226 L 342,222 L 330,228 L 324,242 L 328,255 L 342,264 L 358,266 L 374,264 L 392,268 L 405,274 L 392,286 L 376,294 L 358,304 L 342,312 L 332,324 L 338,340 L 348,354 L 362,366 L 374,376 L 360,384 L 352,392 L 364,398 L 368,412 L 360,428 L 350,444 L 336,460 L 322,474 L 304,488 L 286,502 L 265,512 L 248,524 L 242,536 L 228,532 L 208,538 L 185,546 L 160,555"/>
                        </g>

                        <g fill="rgba(148, 163, 184, 0.42)" font-size="9" font-weight="700" text-anchor="middle" pointer-events="none">
                            <text id="sea-bohai" x="358" y="244">발해만</text>
                            <text id="sea-west" x="422" y="335">서해</text>
                            <text id="sea-east" x="618" y="288">동해</text>
                            <text id="sea-strait" x="540" y="405">대한해협</text>
                        </g>

                        <g id="labels-layer" pointer-events="none"></g>
                    </svg>
                </div>
            </section>

            <!-- [하단 3] 60초 타이머 & 행동 중심 간결한 4개 선택지 -->
            <section class="rounded-xl bg-[#0d131d] border border-white/10 p-2.5 flex flex-col justify-between shrink-0 shadow-lg">
                <div id="timer-control-bar" class="mb-2 px-2.5 py-1.5 rounded-lg bg-[#080c13] border border-white/10">
                    <div class="flex items-center justify-between gap-2 mb-1">
                        <div class="flex items-center gap-1.5">
                            <span class="w-1.5 h-1.5 rounded-full bg-amber-400 animate-ping" id="timer-dot"></span>
                            <span id="lbl-timer-title" class="text-[10px] font-bold text-gray-300">결단 시간 (1턴=10년)</span>
                            <span id="timer-sec-text" class="text-xs font-extrabold text-amber-400 tabular-nums">60.0초</span>
                        </div>

                        <button onclick="cycleGameSpeed()" id="btn-speed-toggle" class="px-2 py-0.5 rounded bg-amber-500/20 hover:bg-amber-500/30 active:scale-95 border border-amber-400/50 text-amber-300 text-[10px] font-extrabold transition flex items-center gap-1">
                            <i class="fa-solid fa-forward-fast text-[9px]"></i>
                            <span id="speed-label">1배속</span>
                        </button>
                    </div>
                    <div class="w-full h-1 bg-gray-800 rounded-full overflow-hidden">
                        <div id="timer-progress-bar" class="h-full bg-gradient-to-r from-emerald-400 via-amber-400 to-rose-500 transition-all duration-100" style="width: 100%;"></div>
                    </div>
                </div>

                <div class="mb-2">
                    <div class="flex items-center justify-between gap-1 mb-0.5">
                        <span id="ev-tag" class="px-1.5 py-0.2 rounded bg-amber-500/15 border border-amber-500/30 text-amber-300 text-[9px] font-bold">
                            정무 전략
                        </span>
                        <span id="ev-turn-info" class="text-[9px] text-gray-400 font-semibold">
                            제1턴 / 60턴
                        </span>
                    </div>
                    <h2 id="ev-title" class="serif-kr text-xs sm:text-sm font-bold text-white leading-snug break-words">
                        사건 제목
                    </h2>
                    <p id="ev-desc" class="text-[11px] text-gray-300 leading-snug mt-0.5 break-words">
                        사건 설명
                    </p>
                </div>

                <!-- 행동 중심 간결한 선택지 버튼 영역 (행동 요약 + 성공 확률만 표시) -->
                <div id="choices-box" class="space-y-1.5"></div>

                <div id="ending-box" class="hidden space-y-2 pt-1.5 border-t border-white/10">
                    <div class="p-2.5 rounded-xl bg-black/60 border border-amber-500/40">
                        <div id="end-badge" class="text-[10px] font-extrabold text-amber-400 mb-0.5">역사의 결말</div>
                        <h3 id="end-title" class="serif-kr text-sm font-bold text-white mb-1">결말 제목</h3>
                        <p id="end-desc" class="text-[11px] text-gray-300 leading-snug mb-1.5 break-words"></p>
                        <div id="end-summary" class="text-[10px] text-amber-200/90 bg-white/5 rounded-lg p-1.5 border border-white/10"></div>
                    </div>
                    <button onclick="initNewRandomGame()" id="btn-ending-restart" class="w-full py-2.5 rounded-xl bg-amber-500 hover:bg-amber-400 text-black font-extrabold text-xs transition shadow-lg">
                        새로운 무작위 왕조로 다시 시작
                    </button>
                </div>
            </section>

        </main>

        <!-- 3. 결과 창 상세 분리 팝업 모달 (선택 직후 부연 설명 및 국력·영토 변동 상세 표시) -->
        <div id="result-popup-overlay" class="hidden absolute inset-0 z-50 bg-black/85 backdrop-blur-sm flex items-center justify-center p-3">
            <div class="modal-animate w-full rounded-2xl bg-[#0e1522] border border-white/15 shadow-2xl overflow-y-auto max-h-[90dvh] p-4 space-y-3">
                <div id="popup-header-bar" class="px-3 py-2 rounded-xl flex items-center justify-between border">
                    <div class="flex items-center gap-1.5">
                        <span id="popup-outcome-badge" class="px-2.5 py-0.5 rounded text-xs font-extrabold">성공</span>
                        <span id="popup-prob-detail" class="text-[11px] text-gray-200 font-semibold">성공 확률 75%</span>
                    </div>
                    <span id="popup-turn-label" class="text-[11px] text-amber-300 font-bold">서기 391년</span>
                </div>

                <div>
                    <div class="text-[10px] font-bold text-gray-400 mb-0.5">결단 및 상세 실행 결과</div>
                    <h3 id="popup-main-title" class="serif-kr text-sm font-bold text-white mb-1 leading-snug break-words">결단 결과 제목</h3>
                    <p id="popup-main-desc" class="text-xs text-gray-200 leading-relaxed break-words"></p>
                </div>

                <div class="p-3 rounded-xl bg-black/50 border border-white/10 space-y-2">
                    <div class="text-[10px] font-bold text-gray-400">국력 및 60분획 영토 변동 내역</div>
                    <div id="popup-stat-chips" class="flex flex-wrap gap-1"></div>
                    <div id="popup-terr-change" class="text-[11px] font-semibold text-amber-300 break-words"></div>
                </div>

                <div class="p-3 rounded-xl bg-amber-500/10 border border-amber-500/30 space-y-1">
                    <div class="flex items-center justify-between">
                        <span class="text-[10px] font-extrabold text-amber-300 flex items-center gap-1">
                            <i class="fa-solid fa-bolt"></i>
                            <span id="popup-chain-tag">연쇄 발생 사건</span>
                        </span>
                        <span id="popup-chain-eff" class="text-[10px] font-bold text-emerald-300"></span>
                    </div>
                    <h4 id="popup-chain-title" class="text-xs font-bold text-white break-words">파생 사건 제목</h4>
                    <p id="popup-chain-desc" class="text-[11px] text-gray-300 leading-snug break-words"></p>
                </div>

                <button onclick="closeResultPopupAndAdvance()" id="btn-popup-confirm" class="w-full py-3 rounded-xl bg-amber-500 hover:bg-amber-400 active:scale-95 text-black font-extrabold text-xs transition flex items-center justify-center gap-1.5 shadow-lg">
                    <span id="lbl-popup-btn">결과 확인 및 다음 10년 정무로 진행</span>
                    <i class="fa-solid fa-arrow-right"></i>
                </button>
            </div>
        </div>

        <!-- 4. 새로운 국호 선포 및 직접 입력 모달 (영토 30+ 또는 외교 100 도달 시) -->
        <div id="empire-modal-overlay" class="hidden absolute inset-0 z-50 bg-black/90 backdrop-blur-md flex items-center justify-center p-4">
            <div class="modal-animate w-full max-w-sm rounded-3xl bg-[#111827] border-2 border-amber-500/60 p-5 space-y-4 shadow-2xl">
                <div class="text-center">
                    <span class="px-3 py-1 rounded-full bg-amber-500/20 text-amber-300 text-[11px] font-extrabold border border-amber-400/40">
                        대제국 선포의 날
                    </span>
                    <h2 class="serif-kr text-lg font-black text-amber-200 mt-2">새로운 국호를 선포하십시오</h2>
                    <p class="text-xs text-gray-300 mt-1 break-words">
                        광대한 영토와 높은 외교 영향력을 달성하여 천하를 호령할 대제국의 새로운 이름을 선포할 기회가 도래했습니다.
                    </p>
                </div>

                <div class="space-y-2">
                    <label class="text-[11px] font-bold text-gray-400">추천 국호 선택</label>
                    <div id="empire-preset-chips" class="grid grid-cols-2 gap-2">
                        <button onclick="selectPresetEmpireName('대고구려 제국')" class="p-2 rounded-xl bg-white/5 hover:bg-amber-500/20 border border-white/10 text-xs font-bold text-amber-300 transition">대고구려 제국</button>
                        <button onclick="selectPresetEmpireName('대백제 해양제국')" class="p-2 rounded-xl bg-white/5 hover:bg-amber-500/20 border border-white/10 text-xs font-bold text-amber-300 transition">대백제 해양제국</button>
                        <button onclick="selectPresetEmpireName('대신라 삼한제국')" class="p-2 rounded-xl bg-white/5 hover:bg-amber-500/20 border border-white/10 text-xs font-bold text-amber-300 transition">대신라 삼한제국</button>
                        <button onclick="selectPresetEmpireName('동방 대제국')" class="p-2 rounded-xl bg-white/5 hover:bg-amber-500/20 border border-white/10 text-xs font-bold text-amber-300 transition">동방 대제국</button>
                    </div>
                </div>

                <div class="space-y-1">
                    <label class="text-[11px] font-bold text-gray-400">직접 국호 입력</label>
                    <input type="text" id="custom-empire-input" placeholder="원하는 제국 이름을 입력하세요" class="w-full px-3 py-2.5 rounded-xl bg-black/60 border border-white/20 text-xs text-white font-bold focus:outline-none focus:border-amber-400">
                </div>

                <button onclick="confirmNewEmpireName()" class="w-full py-3 rounded-xl bg-gradient-to-r from-amber-500 to-orange-500 text-gray-950 font-black text-xs transition shadow-lg">
                    새로운 국호로 천하에 공포하기
                </button>
            </div>
        </div>

    </div>
    <!-- 스마트폰 9:16 세로형 컨테이너 끝 -->

    <script>
        /* =========================================================================
           0. 다국어 설정 및 2배 HD 도트 일러스트 (640x360) 데이터
           ========================================================================= */
        let currentLang = 'ko';
        let selectedIntroSceneIndex = 0;
        let isIntroActive = true;

        const INTRO_PIXEL_SCENES = [
            { id: 1, ko: '2배 HD 도트 · 고구려 개마무사 철기병 대돌격', en: '2x HD Pixel · Goguryeo Iron Cavalry Charge', skyTop: '#1a060a', skyBot: '#450a0a', ground: '#27171a', accent: '#ef4444' },
            { id: 2, ko: '2배 HD 도트 · 백제 황포돛배 대선단과 왕실', en: '2x HD Pixel · Baekje Grand Fleet & Royal Court', skyTop: '#081c24', skyBot: '#0f3f52', ground: '#0369a1', accent: '#f59e0b' },
            { id: 3, ko: '2배 HD 도트 · 신라 서라벌 달빛 화랑도 궁수대', en: '2x HD Pixel · Silla Elite Archers under Moonlight', skyTop: '#041a14', skyBot: '#064e3b', ground: '#11261f', accent: '#10b981' },
            { id: 4, ko: '2배 HD 도트 · 한강 유역 삼국 군주 정상 회동', en: '2x HD Pixel · Summit of Three Monarchs at Han River', skyTop: '#1e1b4b', skyBot: '#4338ca', ground: '#1e293b', accent: '#fbbf24' },
            { id: 5, ko: '2배 HD 도트 · 요동 안시성 성벽 결사항전', en: '2x HD Pixel · Fortress Defense at Ansi Castle', skyTop: '#2a0808', skyBot: '#7f1d1d', ground: '#292524', accent: '#f87171' }
        ];

        function pickRandomIntroScene() {
            let nextIdx = Math.floor(Math.random() * INTRO_PIXEL_SCENES.length);
            if (INTRO_PIXEL_SCENES.length > 1 && nextIdx === selectedIntroSceneIndex) {
                nextIdx = (nextIdx + 1) % INTRO_PIXEL_SCENES.length;
            }
            selectedIntroSceneIndex = nextIdx;
            updateIntroSceneBadge();
        }

        function updateIntroSceneBadge() {
            const sc = INTRO_PIXEL_SCENES[selectedIntroSceneIndex];
            const el = document.getElementById('intro-scene-badge');
            if (el && sc) {
                el.textContent = currentLang === 'ko' ? sc.ko : sc.en;
            }
        }

        function startGameFromIntro() {
            isIntroActive = false;
            document.getElementById('intro-title-screen').classList.add('hidden');
            state.lastFrameTime = performance.now();
        }

        function initVisitorCounter() {
            try {
                const todayKey = new Date().toISOString().slice(0, 10);
                const savedDate = localStorage.getItem('tk_visit_date');
                let dailyCount = parseInt(localStorage.getItem('tk_visit_daily') || '142', 10);
                let totalCount = parseInt(localStorage.getItem('tk_visit_total') || '3891', 10);

                if (savedDate !== todayKey) {
                    dailyCount = 105 + (todayKey.split('-').reduce((a,b)=>a+parseInt(b),0) % 50);
                    localStorage.setItem('tk_visit_date', todayKey);
                }
                if (!sessionStorage.getItem('tk_session_counted')) {
                    dailyCount += 1;
                    totalCount += 1;
                    sessionStorage.setItem('tk_session_counted', '1');
                    localStorage.setItem('tk_visit_daily', String(dailyCount));
                    localStorage.setItem('tk_visit_total', String(totalCount));
                }
                document.getElementById('val-visit-today').textContent = dailyCount.toLocaleString();
                document.getElementById('val-visit-total').textContent = totalCount.toLocaleString();
            } catch (e) {
                document.getElementById('val-visit-today').textContent = '142';
                document.getElementById('val-visit-total').textContent = '3,891';
            }
        }

        /* =========================================================================
           1. 국가 및 군주 데이터
           ========================================================================= */
        const KINGDOMS = {
            goguryeo: {
                id: 'goguryeo', name: '고구려', nameEn: 'Goguryeo', badgeCls: 'bg-red-500/20 text-red-300 border-red-500/40', startYear: 391,
                stats: { economy: 60, military: 74, authority: 62, diplomacy: 54 },
                rulers: [
                    { ko: '고담덕 태왕', en: 'Grand King Go Dam-deok' }, { ko: '고무휼 태왕', en: 'Grand King Go Mu-hyul' }, { ko: '해모루 태왕', en: 'Grand King Hae Mo-ru' }
                ]
            },
            baekje: {
                id: 'baekje', name: '백제', nameEn: 'Baekje', badgeCls: 'bg-amber-500/20 text-amber-300 border-amber-500/40', startYear: 346,
                stats: { economy: 74, military: 58, authority: 58, diplomacy: 68 },
                rulers: [
                    { ko: '부여근초 어라하', en: 'King Buyeo Geun-cho' }, { ko: '부여무령 어라하', en: 'King Buyeo Mu-ryeong' }, { ko: '온조달 어라하', en: 'King Onjo-dal' }
                ]
            },
            silla: {
                id: 'silla', name: '신라', nameEn: 'Silla', badgeCls: 'bg-emerald-500/20 text-emerald-300 border-emerald-500/40', startYear: 500,
                stats: { economy: 62, military: 60, authority: 72, diplomacy: 68 },
                rulers: [
                    { ko: '김이사부 마립간', en: 'Maripgan Kim Isabu' }, { ko: '김마루 마립간', en: 'Maripgan Kim Maru' }, { ko: '김진흥 태왕', en: 'Grand King Kim Jin-heung' }
                ]
            }
        };

        const FACTION_META = {
            goguryeo: { name: '고구려', nameEn: 'Goguryeo', fill: '#b91c1c' },
            baekje:   { name: '백제',   nameEn: 'Baekje',   fill: '#d97706' },
            silla:    { name: '신라',   nameEn: 'Silla',    fill: '#047857' },
            china:    { name: '중원제국', nameEn: 'China Empire', fill: '#581c87' },
            nomad:    { name: '북방초원', nameEn: 'Northern Steppe', fill: '#334155' },
            wa:       { name: '왜/류큐', nameEn: 'Wa/Ryukyu',  fill: '#0e7490' }
        };

        /* =========================================================================
           2. 실제 해안선 기반 60분획 정밀 동아시아 지도 데이터
           ========================================================================= */
        const PROVINCES_60 = [
            // --- [한반도: 18개 분획 (k01 ~ k18)] ---
            { id: 'k01', name: '압록평양', nameEn: 'Pyeongyang', zone: '한반도 북부', zoneEn: 'North Korea', owner: 'goguryeo', capital: 'goguryeo', label: [484, 290], path: 'M 450,270 L 510,270 L 510,302 L 450,302 Z' },
            { id: 'k02', name: '개마고원', nameEn: 'Gaema Plateau', zone: '한반도 북부', zoneEn: 'North Korea', owner: 'goguryeo', label: [524, 260], path: 'M 505,240 L 555,240 L 555,278 L 510,278 L 510,270 L 505,270 Z' },
            { id: 'k03', name: '신의주', nameEn: 'Sinuiju', zone: '한반도 북서', zoneEn: 'NW Korea', owner: 'goguryeo', label: [438, 274], path: 'M 418,255 L 454,255 L 454,295 L 418,295 Z' },
            { id: 'k04', name: '황해대방', nameEn: 'Hwanghae', zone: '한반도 중부', zoneEn: 'Central Korea', owner: 'goguryeo', label: [484, 314], path: 'M 455,302 L 512,302 L 512,320 L 455,320 Z' },
            { id: 'k05', name: '원산안변', nameEn: 'Anbyeon', zone: '한반도 동부', zoneEn: 'East Korea', owner: 'goguryeo', label: [534, 294], path: 'M 510,278 L 560,278 L 560,314 L 512,314 L 512,302 L 510,302 Z' },
            { id: 'k06', name: '함흥북청', nameEn: 'Hamheung', zone: '한반도 동북', zoneEn: 'NE Korea', owner: 'goguryeo', label: [565, 255], path: 'M 555,240 L 600,240 L 600,285 L 555,285 Z' },
            { id: 'k07', name: '한성도읍', nameEn: 'Hanseong', zone: '한반도 중부', zoneEn: 'Central Korea', owner: 'baekje', capital: 'baekje', label: [492, 332], path: 'M 465,320 L 518,320 L 518,342 L 465,342 Z' },
            { id: 'k08', name: '강원춘천', nameEn: 'Chuncheon', zone: '한반도 동부', zoneEn: 'East Korea', owner: 'silla', label: [538, 330], path: 'M 512,314 L 585,314 L 585,342 L 518,342 L 518,320 L 512,320 Z' },
            { id: 'k09', name: '경기인천', nameEn: 'Incheon', zone: '한반도 서부', zoneEn: 'West Korea', owner: 'baekje', label: [445, 330], path: 'M 425,315 L 465,315 L 465,350 L 425,350 Z' },
            { id: 'k10', name: '웅진사비', nameEn: 'Ungjin-Sabi', zone: '한반도 서남', zoneEn: 'SW Korea', owner: 'baekje', label: [492, 354], path: 'M 465,342 L 520,342 L 520,364 L 465,364 Z' },
            { id: 'k11', name: '충주청주', nameEn: 'Chungju', zone: '한반도 중남', zoneEn: 'Central-South Korea', owner: 'silla', label: [528, 348], path: 'M 520,342 L 550,342 L 550,364 L 520,364 Z' },
            { id: 'k12', name: '마한나주', nameEn: 'Naju', zone: '한반도 서남', zoneEn: 'SW Korea', owner: 'baekje', label: [472, 380], path: 'M 452,364 L 500,364 L 500,392 L 452,392 Z' },
            { id: 'k13', name: '완도해상', nameEn: 'Wando Sea', zone: '남해 도서', zoneEn: 'Southern Island', owner: 'baekje', label: [512, 386], path: 'M 500,364 L 532,364 L 532,392 L 500,392 Z' },
            { id: 'k14', name: '탐라제주', nameEn: 'Tamna Jeju', zone: '남해 거점', zoneEn: 'Jeju Island', owner: 'baekje', isIsland: true, label: [472, 434], path: 'M 456,426 L 488,426 L 492,436 L 478,442 L 454,438 Z' },
            { id: 'k15', name: '전주완산', nameEn: 'Jeonju', zone: '한반도 서남', zoneEn: 'SW Korea', owner: 'baekje', label: [508, 368], path: 'M 495,354 L 525,354 L 525,375 L 495,375 Z' },
            { id: 'k16', name: '가야금관', nameEn: 'Geumgwan Gaya', zone: '한반도 남부', zoneEn: 'South Korea', owner: 'silla', label: [536, 376], path: 'M 525,364 L 548,364 L 548,388 L 525,388 Z' },
            { id: 'k17', name: '상주사벌', nameEn: 'Sangju', zone: '한반도 동남', zoneEn: 'SE Korea', owner: 'silla', label: [548, 352], path: 'M 538,342 L 565,342 L 565,364 L 538,364 Z' },
            { id: 'k18', name: '서라벌금성', nameEn: 'Seorabeol', zone: '한반도 동남', zoneEn: 'SE Korea', owner: 'silla', capital: 'silla', label: [564, 370], path: 'M 548,354 L 584,354 L 584,386 L 548,386 Z' },

            // --- [만주 및 북방 초원: 14개 분획 (m01 ~ m14)] ---
            { id: 'm01', name: '요동성', nameEn: 'Liaodong', zone: '만주 서부', zoneEn: 'West Manchuria', owner: 'goguryeo', label: [412, 232], path: 'M 382,210 L 442,210 L 442,255 L 382,255 Z' },
            { id: 'm02', name: '안시성', nameEn: 'Ansi Fortress', zone: '요동 반도', zoneEn: 'Liaodong Pen.', owner: 'goguryeo', label: [400, 260], path: 'M 378,248 L 424,248 L 424,276 L 378,276 Z' },
            { id: 'm03', name: '국내성환도', nameEn: 'Gungnae', zone: '압록 중류', zoneEn: 'Mid Yalu', owner: 'goguryeo', label: [474, 244], path: 'M 442,222 L 505,222 L 505,270 L 442,270 Z' },
            { id: 'm04', name: '부여용담', nameEn: 'Buyeo', zone: '만주 중부', zoneEn: 'Central Manchuria', owner: 'goguryeo', label: [488, 196], path: 'M 442,170 L 532,170 L 532,222 L 442,222 Z' },
            { id: 'm05', name: '백두성산', nameEn: 'Mt. Baekdu', zone: '백두 산맥', zoneEn: 'Baekdu Range', owner: 'goguryeo', label: [544, 224], path: 'M 505,205 L 582,205 L 582,240 L 505,240 Z' },
            { id: 'm06', name: '책성동부여', nameEn: 'Dongbuyeo', zone: '두만 유역', zoneEn: 'Tumen Basin', owner: 'goguryeo', label: [604, 212], path: 'M 582,185 L 642,185 L 642,240 L 582,240 Z' },
            { id: 'm07', name: '연해주해안', nameEn: 'Primorsky', zone: '연해주', zoneEn: 'Maritime Prov.', owner: 'nomad', label: [664, 162], path: 'M 625,125 L 715,125 L 715,195 L 625,195 Z' },
            { id: 'm08', name: '송화강하류', nameEn: 'Lower Songhua', zone: '북만주', zoneEn: 'North Manchuria', owner: 'nomad', label: [568, 146], path: 'M 515,110 L 625,110 L 625,185 L 515,185 Z' },
            { id: 'm09', name: '흑룡강유역', nameEn: 'Amur Basin', zone: '극동 북방', zoneEn: 'Far North', owner: 'nomad', label: [665, 76], path: 'M 580,25 L 780,25 L 780,125 L 580,125 Z' },
            { id: 'm10', name: '흥안령산맥', nameEn: 'Khingan Range', zone: '서북 만주', zoneEn: 'NW Manchuria', owner: 'nomad', label: [446, 122], path: 'M 380,75 L 515,75 L 515,170 L 380,170 Z' },
            { id: 'm11', name: '거란요하', nameEn: 'Khitan Liao', zone: '요서 초원', zoneEn: 'West Liao', owner: 'nomad', label: [358, 184], path: 'M 310,150 L 382,150 L 382,218 L 310,218 Z' },
            { id: 'm12', name: '돌궐동부', nameEn: 'East Turkic', zone: '몽골 동부', zoneEn: 'East Steppe', owner: 'nomad', label: [305, 102], path: 'M 220,45 L 380,45 L 380,150 L 220,150 Z' },
            { id: 'm13', name: '고비사막남단', nameEn: 'South Gobi', zone: '초원 남부', zoneEn: 'South Steppe', owner: 'nomad', label: [185, 148], path: 'M 110,100 L 245,100 L 245,195 L 110,195 Z' },
            { id: 'm14', name: '돌궐아정', nameEn: 'Turkic Royal Court', zone: '몽골 고원', zoneEn: 'Steppe Core', owner: 'nomad', capital: 'nomad', label: [115, 62], path: 'M 10,15 L 220,15 L 220,100 L 10,100 Z' },

            // --- [중원 대륙: 18개 분획 (c01 ~ c18)] ---
            { id: 'c01', name: '요서산해관', nameEn: 'Liaoxi Pass', zone: '하북 동북', zoneEn: 'NE Hebei', owner: 'china', label: [348, 224], path: 'M 310,205 L 382,205 L 382,245 L 310,245 Z' },
            { id: 'c02', name: '유주북평', nameEn: 'Youzhou Beiping', zone: '하북 북부', zoneEn: 'North Hebei', owner: 'china', label: [286, 226], path: 'M 245,195 L 325,195 L 325,258 L 245,258 Z' },
            { id: 'c03', name: '업성기주', nameEn: 'Jizhou Ye', zone: '하북 평원', zoneEn: 'Hebei Plain', owner: 'china', label: [282, 282], path: 'M 240,258 L 330,258 L 330,308 L 240,308 Z' },
            { id: 'c04', name: '산둥등주', nameEn: 'Dengzhou', zone: '산둥 반도', zoneEn: 'Shandong Pen.', owner: 'china', label: [372, 286], path: 'M 342,268 L 408,268 L 408,306 L 342,306 Z' },
            { id: 'c05', name: '제남청주', nameEn: 'Jinan Qingzhou', zone: '산둥 내륙', zoneEn: 'Inland Shandong', owner: 'china', label: [332, 316], path: 'M 295,295 L 368,295 L 368,338 L 295,338 Z' },
            { id: 'c06', name: '태원병주', nameEn: 'Taiyuan', zone: '산서 분지', zoneEn: 'Shanxi Basin', owner: 'china', label: [212, 254], path: 'M 170,210 L 245,210 L 245,295 L 170,295 Z' },
            { id: 'c07', name: '삭방오르도스', nameEn: 'Ordos', zone: '하투 평원', zoneEn: 'Hetao Loop', owner: 'china', label: [126, 226], path: 'M 75,185 L 170,185 L 170,268 L 75,268 Z' },
            { id: 'c08', name: '양주하서회랑', nameEn: 'Hexi Corridor', zone: '실크로드 관문', zoneEn: 'Silk Road Gate', owner: 'china', label: [48, 228], path: 'M 5,175 L 75,175 L 75,280 L 5,280 Z' },
            { id: 'c09', name: '장안관중', nameEn: 'Chang-an', zone: '관중 분지', zoneEn: 'Guanzhong', owner: 'china', capital: 'china', label: [152, 322], path: 'M 100,285 L 200,285 L 200,355 L 100,355 Z' },
            { id: 'c10', name: '낙양중원', nameEn: 'Luoyang', zone: '중원 핵심', zoneEn: 'Central Plains', owner: 'china', label: [242, 326], path: 'M 200,295 L 295,295 L 295,355 L 200,355 Z' },
            { id: 'c11', name: '서주회수', nameEn: 'Xuzhou Huai', zone: '회하 유역', zoneEn: 'Huai Basin', owner: 'china', label: [334, 358], path: 'M 295,338 L 374,338 L 374,380 L 295,380 Z' },
            { id: 'c12', name: '양주강도', nameEn: 'Yangzhou', zone: '강북 운하', zoneEn: 'Grand Canal', owner: 'china', label: [342, 396], path: 'M 305,380 L 374,380 L 374,415 L 305,415 Z' },
            { id: 'c13', name: '건강강남', nameEn: 'Jiankang', zone: '강남 삼각주', zoneEn: 'Yangtze Delta', owner: 'china', label: [338, 434], path: 'M 295,415 L 374,415 L 374,458 L 295,458 Z' },
            { id: 'c14', name: '형주양양', nameEn: 'Jingzhou', zone: '장강 중류', zoneEn: 'Mid Yangtze', owner: 'china', label: [236, 392], path: 'M 185,355 L 295,355 L 295,430 L 185,430 Z' },
            { id: 'c15', name: '성도촉한', nameEn: 'Chengdu Shu', zone: '사천 분지', zoneEn: 'Sichuan Basin', owner: 'china', label: [114, 396], path: 'M 40,355 L 185,355 L 185,445 L 40,445 Z' },
            { id: 'c16', name: '항주월주', nameEn: 'Hangzhou', zone: '절강 해안', zoneEn: 'Zhejiang Coast', owner: 'china', label: [322, 478], path: 'M 275,458 L 358,458 L 358,500 L 275,500 Z' },
            { id: 'c17', name: '복주천주', nameEn: 'Fuzhou', zone: '동남 해안', zoneEn: 'SE Coast', owner: 'china', label: [282, 516], path: 'M 235,500 L 325,500 L 325,542 L 235,542 Z' },
            { id: 'c18', name: '광주영남', nameEn: 'Guangzhou', zone: '영남 남해', zoneEn: 'Lingnan South', owner: 'china', label: [176, 504], path: 'M 95,445 L 275,445 L 275,552 L 95,552 Z' },

            // --- [왜 열도 및 류큐: 10개 분획 (w01 ~ w10)] ---
            { id: 'w01', name: '대마도', nameEn: 'Tsushima', zone: '대한해협', zoneEn: 'Korea Strait', owner: 'wa', isIsland: true, label: [565, 412], path: 'M 556,402 L 574,402 L 572,422 L 554,420 Z' },
            { id: 'w02', name: '북규슈축자', nameEn: 'North Kyushu', zone: '규슈 북부', zoneEn: 'North Kyushu', owner: 'wa', isIsland: true, label: [574, 446], path: 'M 552,432 L 596,432 L 594,460 L 550,458 Z' },
            { id: 'w03', name: '남규슈구마소', nameEn: 'South Kyushu', zone: '규슈 남부', zoneEn: 'South Kyushu', owner: 'wa', isIsland: true, label: [572, 478], path: 'M 552,460 L 592,460 L 586,496 L 550,492 Z' },
            { id: 'w04', name: '주고쿠출운', nameEn: 'Izumo Chugoku', zone: '혼슈 서부', zoneEn: 'West Honshu', owner: 'wa', isIsland: true, label: [622, 430], path: 'M 598,416 L 648,412 L 646,442 L 598,444 Z' },
            { id: 'w05', name: '시코쿠사누키', nameEn: 'Shikoku', zone: '세토 내해', zoneEn: 'Shikoku Isl.', owner: 'wa', isIsland: true, label: [628, 462], path: 'M 606,448 L 650,446 L 648,474 L 604,474 Z' },
            { id: 'w06', name: '아스카야마토', nameEn: 'Asuka Yamato', zone: '기나이 왕도', zoneEn: 'Kinai Core', owner: 'wa', capital: 'wa', isIsland: true, label: [672, 434], path: 'M 648,412 L 696,406 L 696,452 L 648,452 Z' },
            { id: 'w07', name: '주부오와리', nameEn: 'Owari Chubu', zone: '혼슈 중부', zoneEn: 'Central Honshu', owner: 'wa', isIsland: true, label: [716, 412], path: 'M 696,390 L 738,384 L 738,434 L 696,438 Z' },
            { id: 'w08', name: '간토무사시', nameEn: 'Kanto Musashi', zone: '간토 평야', zoneEn: 'Kanto Plain', owner: 'wa', isIsland: true, label: [758, 392], path: 'M 738,368 L 784,362 L 782,414 L 738,418 Z' },
            { id: 'w09', name: '도호쿠에미시', nameEn: 'Tohoku Emishi', zone: '혼슈 북부', zoneEn: 'North Honshu', owner: 'wa', isIsland: true, label: [766, 336], path: 'M 746,304 L 788,298 L 784,362 L 742,368 Z' },
            { id: 'w10', name: '류큐오키나와', nameEn: 'Ryukyu Kingdom', zone: '남해 열도', zoneEn: 'Ryukyu Arc', owner: 'wa', isIsland: true, label: [524, 526], path: 'M 504,514 L 544,512 L 540,538 L 500,536 Z' }
        ];

        /* =========================================================================
           3. 15~20턴 주기 불가항력 역사 통계 시나리오 (개별 확률 5~9.5%)
           ========================================================================= */
        const FORCE_MAJEURE_EVENTS = [
            {
                id: 'fm_iceage', prob: '8.5%', type: 'catastrophe',
                tagKo: '불가항력 자연재해 · 발생 확률 8.5%', tagEn: 'Force Majeure Disaster · Prob 8.5%',
                titleKo: '동아시아 소빙하기 도래와 경신대기근급 한파', titleEn: 'Little Ice Age & Great Famine across East Asia',
                descKo: '오랜 기후 주기 변동으로 여름에도 서리가 내리고 오곡이 얼어 죽는 소빙하기가 도래했습니다. 온 천하가 굶주림에 신음합니다.',
                descEn: 'A sudden Little Ice Age brings summer frost and crop failure across East Asia.',
                choices: [
                    { actKo: '국가 비축미를 전면 개방하고 구휼소 설치', actEn: 'Open All Royal Granaries for Relief', baseProb: 82, eff: { economy: -14, authority: +12, military: -4 }, loseCnt: 0 },
                    { actKo: '남방 온난 곡창지대로 대대적 식량 원정 단행', actEn: 'Launch Grain Expedition to Southern Lands', baseProb: 60, eff: { economy: +10, military: -10, diplomacy: -8 }, gainZone: ['c12', 'c13', 'k12'], gainCnt: 2, loseCnt: 1 },
                    { actKo: '유민들을 북방 둔전과 광산 개간에 강제 동원', actEn: 'Mobilize Refugees for Northern Farming', baseProb: 68, eff: { economy: +8, authority: -14, military: +4 }, gainCnt: 1, loseCnt: 1 },
                    { actKo: '해상 교역선단을 총동원해 동남아 곡물 긴급 수입', actEn: 'Import Emergency Grain via Maritime Fleet', baseProb: 75, eff: { economy: -10, diplomacy: +10, authority: +6 }, gainCnt: 1, loseCnt: 0 }
                ]
            },
            {
                id: 'fm_baekdu', prob: '5.5%', type: 'catastrophe',
                tagKo: '불가항력 지질재앙 · 발생 확률 5.5%', tagEn: 'Volcanic Catastrophe · Prob 5.5%',
                titleKo: '백두산 천지 화산 대분화와 화산재의 겨울', titleEn: 'Millennium Eruption of Mt. Baekdu Volcano',
                descKo: '백두산이 하늘을 찢는 굉음과 함께 폭발하여 화산재가 한반도와 만주, 왜 열도까지 뒤덮었습니다.',
                descEn: 'Mt. Baekdu erupts violently, blanketing Manchuria, Korea, and Japan in volcanic ash.',
                choices: [
                    { actKo: '피해 지역 백성을 도성 이남으로 긴급 이주', actEn: 'Evacuate Citizens South of the Ash Zone', baseProb: 80, eff: { economy: -12, authority: +10, military: -6 }, loseCnt: 0 },
                    { actKo: '혼란에 빠진 주변국 국경 성채를 기습 점령', actEn: 'Blitz Neighboring Fortresses Amid Chaos', baseProb: 58, eff: { military: +8, diplomacy: -12, economy: +6 }, gainCnt: 3, loseCnt: 1 },
                    { actKo: '천제단에 제사를 올리고 왕실 내탕금 하사', actEn: 'Perform Grand Ritual & Donate Royal Gold', baseProb: 85, eff: { authority: +14, economy: -10 }, gainCnt: 1, loseCnt: 0 },
                    { actKo: '화산재 비옥토를 활용한 신농법 개간 사업 추진', actEn: 'Plow Volcanic Ash Soil for New Farmland', baseProb: 72, eff: { economy: +14, authority: +4, military: -4 }, gainCnt: 1, loseCnt: 0 }
                ]
            },
            {
                id: 'fm_plague', prob: '7.2%', type: 'catastrophe',
                tagKo: '불가항력 역병 · 발생 확률 7.2%', tagEn: 'Pandemic Crisis · Prob 7.2%',
                titleKo: '대륙 교역로를 타고 번진 흑사병급 대역병', titleEn: 'Great Plague Spreading Along Trade Routes',
                descKo: '서역과 중원을 휩쓴 치명적인 역병이 국경 포구와 성문에 당도하여 병사와 백성이 쓰러지고 있습니다.',
                descEn: 'A deadly plague from continental trade routes reaches your ports and fortresses.',
                choices: [
                    { actKo: '국경과 포구를 즉시 봉쇄하고 격리 구역 설치', actEn: 'Seal Borders & Establish Quarantine Zones', baseProb: 84, eff: { diplomacy: -10, economy: -8, authority: +10 }, loseCnt: 0 },
                    { actKo: '전국의 의관과 약초를 총동원해 방역 의서 편찬', actEn: 'Mobilize Physicians & Compile Medical Texts', baseProb: 78, eff: { economy: -12, authority: +16, diplomacy: +6 }, gainCnt: 1, loseCnt: 0 },
                    { actKo: '역병으로 무너진 적국의 무방비 성읍 진군', actEn: 'March into Enemy Cities Weakened by Plague', baseProb: 55, eff: { military: -8, economy: +12, authority: -6 }, gainCnt: 3, loseCnt: 1 },
                    { actKo: '청결령을 반포하고 우물과 상수도를 전면 정비', actEn: 'Enforce Hygiene Decrees & Purify Wells', baseProb: 82, eff: { economy: -6, authority: +12, military: +4 }, gainCnt: 1, loseCnt: 0 }
                ]
            },
            {
                id: 'fm_silkroad', prob: '9.5%', type: 'blessing',
                tagKo: '역사적 행운 주기 · 발생 확률 9.5%', tagEn: 'Historical Golden Age · Prob 9.5%',
                titleKo: '실크로드와 동아시아 해상 무역망의 대황금기', titleEn: 'Golden Age of Silk Road & Maritime Trade',
                descKo: '페르시아·서역 상단과 남해 무역선이 우리 포구로 몰려들며 국고에 금은보화와 선진 문물이 넘쳐납니다.',
                descEn: 'Merchants from Persia and the Southern Seas flock to your ports, bringing immense wealth.',
                choices: [
                    { actKo: '국제 자유 무역항을 개설하고 상업세를 대폭 확충', actEn: 'Open International Free Ports for Tax Boom', baseProb: 90, eff: { economy: +20, diplomacy: +14, authority: +6 }, gainCnt: 2, loseCnt: 0 },
                    { actKo: '막대한 무역 수익으로 정예 철기병과 화포군 증강', actEn: 'Invest Trade Profits into Elite Cavalry', baseProb: 85, eff: { military: +18, economy: +8 }, gainCnt: 3, loseCnt: 0 },
                    { actKo: '서역의 천문학·야금술 장인들을 왕실에 영입', actEn: 'Recruit Western Astronomers & Metallurgists', baseProb: 88, eff: { authority: +14, military: +12, diplomacy: +10 }, gainCnt: 2, loseCnt: 0 },
                    { actKo: '무역 독점권을 무기로 인접 세력을 경제적으로 복속', actEn: 'Subjugate Neighbors via Trade Monopoly', baseProb: 76, eff: { diplomacy: +16, economy: +12 }, gainCnt: 3, loseCnt: 0 }
                ]
            },
            {
                id: 'fm_harvest', prob: '9.0%', type: 'blessing',
                tagKo: '천우신조 대풍년 · 발생 확률 9.0%', tagEn: 'Miraculous Harvest · Prob 9.0%',
                titleKo: '10년 연속 비옥한 기후와 삼한 천하 대풍년', titleEn: 'Decade of Ideal Climate & Bumper Harvests',
                descKo: '가뭄과 홍수 없이 온화한 기후가 10년간 이어지며 전국의 창고마다 곡식이 산처럼 쌓였습니다.',
                descEn: 'Ten years of perfect weather fill every granary across the realm to the brim.',
                choices: [
                    { actKo: '백성의 조세를 감면하고 인구 증식 장려책 반포', actEn: 'Cut Taxes & Promote Population Growth', baseProb: 92, eff: { authority: +18, economy: +12, military: +8 }, gainCnt: 1, loseCnt: 0 },
                    { actKo: '넘치는 군량미를 바탕으로 대륙 정벌 대원정 개시', actEn: 'Launch Grand Continental Expedition', baseProb: 74, eff: { military: +14, economy: -4, authority: +10 }, gainCnt: 4, loseCnt: 1 },
                    { actKo: '식량 부족에 시달리는 주변국에 곡물을 팔아 영토 할양', actEn: 'Trade Surplus Grain for Neighboring Lands', baseProb: 82, eff: { economy: +16, diplomacy: +14 }, gainCnt: 3, loseCnt: 0 },
                    { actKo: '대규모 운하와 도로망을 건설하여 물류 혁명 달성', actEn: 'Construct Grand Canals & Highway Networks', baseProb: 88, eff: { economy: +18, authority: +10 }, gainCnt: 2, loseCnt: 0 }
                ]
            }
        ];

        /* =========================================================================
           4. 20턴 주기 인류 역사적 변곡점 (기술·사상·외교·정치 업적)
           ========================================================================= */
        const TURNING_POINT_EVENTS = {
            20: {
                tagKo: '제20턴 인류 역사적 변곡점 (서기 +200년)', tagEn: 'Turn 20 Historical Turning Point',
                titleKo: '중세 과학기술과 사상의 대혁명 채택', titleEn: 'Embrace Medieval Scientific & Ideological Revolution',
                descKo: '국가의 기틀이 완성되었습니다. 시대를 앞지르는 위대한 문명적 변곡점 하나를 국가 핵심 노선으로 채택하십시오.',
                descEn: 'Choose a civilization-defining breakthrough to propel your empire ahead of its time.',
                choices: [
                    { actKo: '[기술] 흑색화약과 신기전 화포 부대 세계 최초 창설', actEn: '[Tech] Invent Gunpowder & Rocket Artillery Corps', baseProb: 90, eff: { military: +22, authority: +8, economy: -6 }, gainCnt: 3, loseCnt: 0 },
                    { actKo: '[문화] 금속활자 발명 및 국립 대학 인쇄 보급', actEn: '[Culture] Invent Movable Metal Type & Printing', baseProb: 94, eff: { authority: +20, diplomacy: +16, economy: +10 }, gainCnt: 2, loseCnt: 0 },
                    { actKo: '[해양] 나침반과 대형 장갑 범선 건조로 대양 제패', actEn: '[Naval] Compass & Armored Galleons for Ocean Rule', baseProb: 88, eff: { economy: +20, diplomacy: +14, military: +12 }, gainCnt: 3, loseCnt: 0 },
                    { actKo: '[정치] 골품·신분제를 철폐하고 능력주의 과거제 전면 도입', actEn: '[Politics] Abolish Caste & Adopt Meritocratic Exams', baseProb: 86, eff: { authority: +22, economy: +14, military: +10 }, gainCnt: 2, loseCnt: 0 }
                ]
            },
            40: {
                tagKo: '제40턴 인류 역사적 변곡점 (서기 +400년)', tagEn: 'Turn 40 Historical Turning Point',
                titleKo: '대항해시대와 초기 산업혁명의 개막', titleEn: 'Dawn of Age of Discovery & Early Industrialization',
                descKo: '우리 제국의 학자와 항해사들이 기존의 세계관을 뛰어넘는 거대한 변곡점에 도달했습니다.',
                descEn: 'Your scholars and navigators have reached a monumental turning point in human history.',
                choices: [
                    { actKo: '[기술] 증기기관 발명 및 제철·방직 기계화 공장 가동', actEn: '[Tech] Invent Steam Engine & Mechanized Factories', baseProb: 88, eff: { economy: +25, military: +18, authority: +6 }, gainCnt: 3, loseCnt: 0 },
                    { actKo: '[탐험] 태평양 횡단 신대륙 발견 및 글로벌 세계지도 완성', actEn: '[Explore] Discover New World & Complete World Map', baseProb: 85, eff: { diplomacy: +24, economy: +22, authority: +10 }, gainCnt: 4, loseCnt: 0 },
                    { actKo: '[군사] 강선 소총과 철갑함 함대를 편성하여 주변국 압도', actEn: '[Military] Rifled Muskets & Ironclad Fleet', baseProb: 84, eff: { military: +26, diplomacy: +10, economy: -8 }, gainCnt: 4, loseCnt: 0 },
                    { actKo: '[경제] 중앙은행 설립 및 지폐·주식회사 금융 체제 확립', actEn: '[Economy] Central Bank, Paper Currency & Joint-Stock', baseProb: 92, eff: { economy: +28, authority: +12, diplomacy: +12 }, gainCnt: 2, loseCnt: 0 }
                ]
            },
            60: {
                tagKo: '제60턴 인류 역사적 변곡점 (서기 +600년)', tagEn: 'Turn 60 Historical Turning Point',
                titleKo: '근현대 정치 사상과 체제 대변혁', titleEn: 'Modern Political Ideology & Constitutional Shift',
                descKo: '거대해진 제국의 신민들이 새로운 시대의 통치 철학과 정치 체제를 요구하고 있습니다.',
                descEn: 'Citizens of your vast empire demand a new political philosophy for the modern era.',
                choices: [
                    { actKo: '[민주주의] 입헌군주제 헌법 반포 및 시민 의회 창설', actEn: '[Democracy] Constitutional Monarchy & Parliament', baseProb: 90, eff: { authority: +25, diplomacy: +22, economy: +15 }, gainCnt: 3, loseCnt: 0 },
                    { actKo: '[사회주의] 토지·생산수단 국유화 및 만민 평등 분배 혁명', actEn: '[Socialism] Land Nationalization & Equal Distribution', baseProb: 82, eff: { authority: +20, military: +20, economy: +10, diplomacy: -6 }, gainCnt: 4, loseCnt: 0 },
                    { actKo: '[제국주의] 총력전 군국 체제와 동아시아 단일 연방 선포', actEn: '[Imperial] Total Mobilization & East Asian Federation', baseProb: 80, eff: { military: +28, economy: +14, authority: +8 }, gainCnt: 5, loseCnt: 1 },
                    { actKo: '[과학국가] 기술관료제 도입 및 전 국민 의무교육·의료 시행', actEn: '[Technocracy] Universal Education & Science Directorate', baseProb: 92, eff: { economy: +22, authority: +22, diplomacy: +18 }, gainCnt: 3, loseCnt: 0 }
                ]
            },
            80: {
                tagKo: '제80턴 인류 역사적 변곡점 (서기 +800년)', tagEn: 'Turn 80 Historical Turning Point',
                titleKo: '초현대 문명과 인류 미래 패권 완성', titleEn: 'Ultra-Modern Civilization & Global Hegemony',
                descKo: '수백 년의 세월을 거쳐 동아시아를 넘어 세계사의 중심에 우뚝 섰습니다.',
                descEn: 'After centuries of rule, your nation stands at the pinnacle of world civilization.',
                choices: [
                    { actKo: '[첨단] 전기·통신망 구축 및 현대적 산업 제국 완성', actEn: '[Modern] Power Grid, Telecom & Industrial Empire', baseProb: 94, eff: { economy: +25, military: +25, authority: +20, diplomacy: +20 }, gainCnt: 5, loseCnt: 0 },
                    { actKo: '[평화] 세계 평화 기구 창설 및 국제법 주도권 장악', actEn: '[Peace] Found Global Peace Organization & Int. Law', baseProb: 95, eff: { diplomacy: +30, authority: +25, economy: +15 }, gainCnt: 4, loseCnt: 0 },
                    { actKo: '[정복] 최정예 기갑·해군 전력으로 남은 미수복 영토 총진격', actEn: '[Conquest] Final Grand Offensive on Remaining Lands', baseProb: 88, eff: { military: +30, authority: +15 }, gainCnt: 6, loseCnt: 0 },
                    { actKo: '[문화] 세계 공용어 지정 및 인류 문화유산 수도 선포', actEn: '[Culture] Global Lingua Franca & Cultural Capital', baseProb: 94, eff: { authority: +28, diplomacy: +28, economy: +20 }, gainCnt: 4, loseCnt: 0 }
                ]
            }
        };

        /* =========================================================================
           5. 일반 정무 이벤트 풀 (행동 중심 간결 선택지용)
           ========================================================================= */
        const GENERAL_EVENTS = [
            {
                tagKo: '한반도 중원 패권전', tagEn: 'Peninsula Hegemony',
                titleKo: '한강 유역과 나제·여제 국경 요충지 쟁탈전', titleEn: 'Battle for the Strategic Han River Basin',
                descKo: '한반도의 배꼽인 한강 하구와 중부 내륙 성채를 둘러싸고 삼국의 대군이 대치 중입니다.',
                descEn: 'Armies of the Three Kingdoms clash over the vital Han River basin and central fortresses.',
                choices: [
                    { actKo: '정예 기병과 수군을 동시에 투입해 한강 유역 전면 강습', actEn: 'Full Assault on Han River with Cavalry & Navy', baseProb: 68, eff: { military: -6, authority: +12, economy: +10 }, gainZone: ['k07', 'k09', 'k04', 'k08'], gainCnt: 2, loseCnt: 1 },
                    { actKo: '인접국과 비밀 동맹을 맺어 적의 후방 성채를 협공', actEn: 'Forge Secret Alliance for Pincer Attack', baseProb: 76, eff: { diplomacy: -4, military: +6, economy: +8 }, gainZone: ['k10', 'k11', 'k15', 'k17'], gainCnt: 2, loseCnt: 1 },
                    { actKo: '국경 산성을 증축하고 둔전을 넓혀 장기 소모전 대비', actEn: 'Fortify Mountain Castles & Expand Farmland', baseProb: 88, eff: { economy: +10, military: +8, authority: +6 }, gainCnt: 1, loseCnt: 0 },
                    { actKo: '사신을 보내 휴전 협정을 맺고 해상 무역로를 확보', actEn: 'Send Envoys for Truce & Maritime Trade Routes', baseProb: 84, eff: { diplomacy: +14, economy: +12, authority: +4 }, gainCnt: 1, loseCnt: 0 }
                ]
            },
            {
                tagKo: '만주 · 북방 초원 원정', tagEn: 'Manchuria & Steppe Campaign',
                titleKo: '요동 벌판과 북방 유목 가한의 남하 위협', titleEn: 'Liaodong Plains & Northern Nomadic Threat',
                descKo: '요하 너머 거란·말갈·돌궐 기병대가 세력을 규합하며 국경 성벽을 압박하고 있습니다.',
                descEn: 'Khitan, Mohe, and Turkic cavalry gather beyond the Liao River, threatening the frontier.',
                choices: [
                    { actKo: '중장갑 철기병을 출격시켜 요동과 초원 본영을 선제 토벌', actEn: 'Launch Preemptive Strike with Heavy Iron Cavalry', baseProb: 64, eff: { military: -8, authority: +14, economy: +8 }, gainZone: ['m01', 'm02', 'm11', 'm12'], gainCnt: 3, loseCnt: 1 },
                    { actKo: '유목 부족장들에게 비단과 철기를 하사해 번속국으로 편입', actEn: 'Grant Silk & Iron to Subjugate Nomad Chiefs', baseProb: 78, eff: { economy: -8, diplomacy: +16, military: +10 }, gainZone: ['m07', 'm08', 'm10'], gainCnt: 2, loseCnt: 0 },
                    { actKo: '천리장성을 축조하고 봉수대 체계를 완성해 방어력 극대화', actEn: 'Build Great Frontier Wall & Beacon Network', baseProb: 90, eff: { economy: -6, military: +14, authority: +8 }, gainCnt: 1, loseCnt: 0 },
                    { actKo: '초원 가한과 혼인 동맹을 맺고 중원 제국의 북방을 협공', actEn: 'Royal Marriage Alliance with Steppe Khagan', baseProb: 74, eff: { diplomacy: +14, military: +8, economy: +6 }, gainZone: ['c01', 'c02', 'm11'], gainCnt: 2, loseCnt: 1 }
                ]
            },
            {
                tagKo: '중원 대륙 제국과의 대결', tagEn: 'Showdown with Continental China',
                titleKo: '중원 황제의 조공 요구와 산둥·요서 해상 패권', titleEn: 'Chinese Emperor Demands Tribute & Shandong Hegemony',
                descKo: '중원 대륙의 통일 왕조가 오만한 국서를 보내왔습니다. 굴복할 것인가, 대륙으로 진격할 것인가 결단해야 합니다.',
                descEn: 'The Chinese court sends an arrogant decree. Will you negotiate or march into the continent?',
                choices: [
                    { actKo: '대선단으로 발해만을 건너 산둥반도와 장안으로 북벌 감행', actEn: 'Cross Bohai Sea to Conquer Shandong & Chang-an', baseProb: 56, eff: { military: -10, authority: +18, economy: +14 }, gainZone: ['c01', 'c02', 'c04', 'c05', 'c09', 'c10'], gainCnt: 3, loseCnt: 1 },
                    { actKo: '국경 요새에서 청야전술로 적 대군을 유인해 섬멸', actEn: 'Lure & Annihilate Enemy Army with Scorched-Earth', baseProb: 74, eff: { military: +10, authority: +14, economy: -4 }, gainZone: ['c01', 'm01', 'm02'], gainCnt: 2, loseCnt: 1 },
                    { actKo: '강남·서역 세력과 원교근공 외교를 펼쳐 중원을 분할 압박', actEn: 'Form Distant Alliances to Divide the Mainland', baseProb: 80, eff: { diplomacy: +18, economy: +10 }, gainZone: ['c11', 'c12', 'c13'], gainCnt: 2, loseCnt: 0 },
                    { actKo: '실리적 사절단을 파견해 화친을 유지하고 서해 교역권 독점', actEn: 'Send Pragmatic Envoys & Monopolize Yellow Sea Trade', baseProb: 88, eff: { diplomacy: +12, economy: +16, authority: -2 }, gainCnt: 1, loseCnt: 0 }
                ]
            },
            {
                tagKo: '왜 열도 · 류큐 해양 경략', tagEn: 'Wa Archipelago & Ryukyu Campaign',
                titleKo: '대한해협 제해권 장악과 왜 열도 · 류큐 원정', titleEn: 'Command of Korea Strait & Expedition to Wa/Ryukyu',
                descKo: '대마도와 규슈 해역의 왜구 세력을 제압하고 열도 깊숙이 영향력을 확대할 기회입니다.',
                descEn: 'Secure the Korea Strait and project power across Kyushu, Yamato, and Ryukyu.',
                choices: [
                    { actKo: '대형 전함 선단을 출격시켜 대마도·규슈·야마토를 직접 정벌', actEn: 'Deploy Grand Armada to Conquer Kyushu & Yamato', baseProb: 64, eff: { military: -8, economy: +14, authority: +14 }, gainZone: ['w01', 'w02', 'w03', 'w04', 'w06'], gainCnt: 3, loseCnt: 1 },
                    { actKo: '박사와 기술자를 파견하고 열도 주요 거점에 담로 설치', actEn: 'Dispatch Scholars & Establish Damno Governors', baseProb: 82, eff: { diplomacy: +18, economy: +12, authority: +8 }, gainZone: ['w01', 'w02', 'w05', 'w10'], gainCnt: 2, loseCnt: 0 },
                    { actKo: '탐라와 류큐를 잇는 남방 해상 무역 루트 독점 개발', actEn: 'Develop Tamna-Ryukyu Southern Maritime Route', baseProb: 86, eff: { economy: +18, diplomacy: +12 }, gainZone: ['k14', 'w10', 'c17'], gainCnt: 2, loseCnt: 0 },
                    { actKo: '남해안 수군 진영을 강화하고 해적 소탕령 발동', actEn: 'Fortify Southern Naval Bases & Sweep Pirates', baseProb: 90, eff: { military: +12, authority: +10, economy: +6 }, gainCnt: 1, loseCnt: 0 }
                ]
            },
            {
                tagKo: '왕권 강화와 내정 개혁', tagEn: 'Royal Authority & Domestic Reform',
                titleKo: '지방 호족의 사병 혁파와 국가 율령 반포', titleEn: 'Abolish Noble Private Armies & Promulgate Laws',
                descKo: '오랜 전쟁과 세월 속에 비대해진 귀족 가문들이 사병을 거느리고 왕권을 위협하고 있습니다.',
                descEn: 'Powerful aristocratic clans have amassed private armies, challenging royal authority.',
                choices: [
                    { actKo: '귀족 사병을 즉시 해산시키고 중앙군으로 강제 편입', actEn: 'Disband Noble Armies into Royal Central Forces', baseProb: 66, eff: { military: +16, authority: +14, diplomacy: -4 }, gainCnt: 2, loseCnt: 1 },
                    { actKo: '진대법과 구휼 제도를 법제화하여 백성의 지지를 결집', actEn: 'Enact Relief Laws to Rally Popular Support', baseProb: 88, eff: { authority: +18, economy: -6, military: +6 }, gainCnt: 1, loseCnt: 0 },
                    { actKo: '전국 토지를 재측량하고 조세 정의를 확립해 국고 확충', actEn: 'Survey All Lands & Reform Taxation for Treasury', baseProb: 82, eff: { economy: +20, authority: +8 }, gainCnt: 2, loseCnt: 0 },
                    { actKo: '신진 인재와 화랑·경당 청년들을 등용해 정치 세대교체', actEn: 'Promote Young Talents for Political Renewal', baseProb: 86, eff: { diplomacy: +14, authority: +12, economy: +8 }, gainCnt: 2, loseCnt: 0 }
                ]
            }
        ];

        const CHAIN_EVENTS = [
            { koTitle: '국경 성주들의 자발적 귀순', enTitle: 'Voluntary Defection of Border Lords', koDesc: '우리 국세에 감복한 인접 성주들이 성문을 열고 투항했습니다.', enDesc: 'Neighboring fortress lords opened their gates and surrendered.', eff: { authority: +4, diplomacy: +4 } },
            { koTitle: '전리품과 조공 행렬 도착', enTitle: 'Arrival of War Booty & Tribute', koDesc: '수복한 영토에서 막대한 군량과 금은이 도성 창고로 운송되었습니다.', enDesc: 'Vast supplies of grain and gold arrived at the royal capital.', eff: { economy: +6 } },
            { koTitle: '변경 수비대의 전술 혁신', enTitle: 'Tactical Innovation of Frontier Guards', koDesc: '실전을 겪은 장수들이 새로운 진법과 공성 전술을 완성했습니다.', enDesc: 'Veteran commanders perfected new siege and formation tactics.', eff: { military: +6 } }
        ];

        /* =========================================================================
           6. 게임 상태 관리 및 타이머/배속/히스토리 기록
           ========================================================================= */
        let state = {
            kingdom: 'goguryeo',
            customEmpireName: null,
            hasProclaimedEmpire: false,
            rulerObj: null,
            year: 391,
            turn: 1,
            maxTurns: 60,
            stats: { economy: 60, military: 70, authority: 60, diplomacy: 55 },
            owners: {},
            currentEvent: null,
            gameOver: false,
            isPopupOpen: false,
            timeRemaining: 60.0,
            speedMultiplier: 1,
            lastFrameTime: performance.now(),
            historyMilestones: [] // 30초 엔딩 슬라이드쇼용 기록
        };

        let bannerTick = 0;
        let slideshowInterval = null;
        let slideshowStartTime = 0;

        function clamp(v) {
            return Math.max(0, Math.min(100, Math.round(v)));
        }

        function getPlayerDisplayName() {
            if (state.customEmpireName) return state.customEmpireName;
            const k = KINGDOMS[state.kingdom];
            return currentLang === 'ko' ? k.name : k.nameEn;
        }

        function setLanguage(lang) {
            currentLang = lang;
            const isKo = lang === 'ko';
            ['btn-lang-ko', 'btn-intro-lang-ko'].forEach(id => {
                const el = document.getElementById(id);
                if (el) el.className = isKo ? 'px-2 py-0.5 rounded bg-amber-500 text-black transition' : 'px-2 py-0.5 rounded text-gray-400 hover:text-white transition';
            });
            ['btn-lang-en', 'btn-intro-lang-en'].forEach(id => {
                const el = document.getElementById(id);
                if (el) el.className = !isKo ? 'px-2 py-0.5 rounded bg-amber-500 text-black transition' : 'px-2 py-0.5 rounded text-gray-400 hover:text-white transition';
            });

            document.getElementById('intro-main-title').textContent = isKo ? '한반도 삼국통일 시뮬레이터' : 'Three Kingdoms Unification Simulator';
            document.getElementById('intro-sub-desc').textContent = isKo
                ? '고구려 · 백제 · 신라의 군주가 되어 60개 구역으로 세분화된 한반도, 만주, 중원, 왜 열도를 제패하는 1턴=10년 역사 시뮬레이션'
                : 'Rule Goguryeo, Baekje, or Silla across 60 detailed East Asian regions (1 Turn = 10 Years).';
            document.getElementById('lbl-intro-start').textContent = isKo ? '시작하기' : 'START GAME';
            document.getElementById('lbl-visit-today').textContent = isKo ? '일일 방문자' : 'TODAY';
            document.getElementById('lbl-visit-total').textContent = isKo ? '누적 방문자' : 'TOTAL VISITORS';
            document.getElementById('lbl-new-game').textContent = isKo ? '새 운명' : 'New Fate';
            document.getElementById('lbl-stat-eco').textContent = isKo ? '국고' : 'Treasury';
            document.getElementById('lbl-stat-mil').textContent = isKo ? '군사' : 'Military';
            document.getElementById('lbl-stat-aut').textContent = isKo ? '민심' : 'Support';
            document.getElementById('lbl-stat-dip').textContent = isKo ? '외교' : 'Diplomacy';
            document.getElementById('lbl-timer-title').textContent = isKo ? '결단 시간 (1턴=10년)' : 'Decision Timer (10Y/Turn)';

            updateIntroSceneBadge();
            updateSpeedButtonUI();
            renderMapSVG();
            updateUI();
            if (!state.gameOver && state.currentEvent) {
                renderChoicesUI();
            }
        }

        function cycleGameSpeed() {
            state.speedMultiplier = state.speedMultiplier >= 8 ? 1 : state.speedMultiplier + 1;
            updateSpeedButtonUI();
        }

        function updateSpeedButtonUI() {
            const lbl = document.getElementById('speed-label');
            if (lbl) lbl.textContent = currentLang === 'ko' ? `${state.speedMultiplier}배속` : `${state.speedMultiplier}x`;
        }

        function cycleTargetTurns() {
            const goals = [40, 60, 80];
            const idx = goals.indexOf(state.maxTurns);
            state.maxTurns = goals[(idx + 1) % goals.length];
            updateUI();
        }

        /* =========================================================================
           7. 게임 초기화 및 60분획 SVG 지도 렌더링
           ========================================================================= */
        function initNewRandomGame() {
            stopSlideshowPlayer();
            const keys = ['goguryeo', 'baekje', 'silla'];
            const pickedKey = keys[Math.floor(Math.random() * keys.length)];
            const kData = KINGDOMS[pickedKey];
            const pickedRuler = kData.rulers[Math.floor(Math.random() * kData.rulers.length)];

            state.kingdom = pickedKey;
            state.customEmpireName = null;
            state.hasProclaimedEmpire = false;
            state.rulerObj = pickedRuler;
            state.year = kData.startYear;
            state.turn = 1;
            state.stats = { ...kData.stats };
            state.gameOver = false;
            state.isPopupOpen = false;
            state.timeRemaining = 60.0;
            state.lastFrameTime = performance.now();

            state.owners = {};
            PROVINCES_60.forEach(p => { state.owners[p.id] = p.owner; });

            const initTerr = Object.values(state.owners).filter(o => o === state.kingdom).length;
            state.historyMilestones = [
                {
                    sceneType: 'founding',
                    year: state.year,
                    turn: 1,
                    titleKo: `${kData.name} ${pickedRuler.ko} 즉위`,
                    titleEn: `Coronation of ${pickedRuler.en} of ${kData.nameEn}`,
                    descKo: `서기 ${state.year}년, ${initTerr}개 분획의 영토를 바탕으로 동아시아 대장정을 시작했습니다.`,
                    descEn: `In AD ${state.year}, began the grand campaign with ${initTerr} territories.`
                }
            ];

            document.getElementById('result-popup-overlay').classList.add('hidden');
            document.getElementById('empire-modal-overlay').classList.add('hidden');
            document.getElementById('ending-box').classList.add('hidden');
            document.getElementById('choices-box').classList.remove('hidden');
            document.getElementById('timer-control-bar').classList.remove('hidden');

            renderMapSVG();
            updateUI();
            loadNextTurnEvent();
        }

        function renderMapSVG() {
            const provLayer = document.getElementById('provinces-layer');
            const labelLayer = document.getElementById('labels-layer');
            if (!provLayer || !labelLayer) return;
            provLayer.innerHTML = '';
            labelLayer.innerHTML = '';

            const svgEl = document.getElementById('east-asia-map');
            let islandGroup = document.getElementById('islands-layer');
            if (!islandGroup) {
                islandGroup = document.createElementNS('http://www.w3.org/2000/svg', 'g');
                islandGroup.setAttribute('id', 'islands-layer');
                svgEl.insertBefore(islandGroup, labelLayer);
            }
            islandGroup.innerHTML = '';

            PROVINCES_60.forEach(p => {
                const owner = state.owners[p.id];
                const meta = FACTION_META[owner] || FACTION_META.nomad;
                const isPlayer = owner === state.kingdom;

                const path = document.createElementNS('http://www.w3.org/2000/svg', 'path');
                path.setAttribute('id', `prov-${p.id}`);
                path.setAttribute('d', p.path);
                path.setAttribute('fill', meta.fill);
                path.setAttribute('fill-opacity', isPlayer ? '0.92' : '0.62');
                path.setAttribute('class', 'prov-cell');

                if (p.isIsland) {
                    path.style.stroke = '#e2e8f0';
                    path.style.strokeWidth = '1.1px';
                    path.style.strokeDasharray = 'none';
                }

                const showInfo = () => {
                    const infoEl = document.getElementById('map-hover-info');
                    if (infoEl) {
                        infoEl.textContent = currentLang === 'ko'
                            ? `[${p.zone}] ${p.name} · ${meta.name}`
                            : `[${p.zoneEn}] ${p.nameEn} · ${meta.nameEn}`;
                    }
                };
                path.addEventListener('mouseenter', showInfo);
                path.addEventListener('click', showInfo);

                if (p.isIsland) islandGroup.appendChild(path);
                else provLayer.appendChild(path);

                const text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                text.setAttribute('x', p.label[0]);
                text.setAttribute('y', p.label[1]);
                text.setAttribute('text-anchor', 'middle');
                text.setAttribute('dominant-baseline', 'central');
                text.setAttribute('fill', isPlayer ? '#ffffff' : 'rgba(226,232,240,0.78)');
                text.setAttribute('font-size', p.capital ? '7.5' : '6.3');
                text.setAttribute('font-weight', isPlayer || p.capital ? '800' : '600');
                text.style. paintOrder = 'stroke';
                text.style.stroke = 'rgba(2,6,23,0.92)';
                text.style.strokeWidth = '2px';
                const dispName = currentLang === 'ko' ? p.name : p.nameEn;
                text.textContent = p.capital ? `★${dispName}` : dispName;
                labelLayer.appendChild(text);
            });
        }

        function updateMapColors() {
            const counts = { goguryeo: 0, baekje: 0, silla: 0, china: 0, nomad: 0, wa: 0 };
            PROVINCES_60.forEach(p => {
                const owner = state.owners[p.id];
                counts[owner] = (counts[owner] || 0) + 1;
                const el = document.getElementById(`prov-${p.id}`);
                if (el) {
                    const meta = FACTION_META[owner];
                    const isPlayer = owner === state.kingdom;
                    el.setAttribute('fill', meta.fill);
                    el.setAttribute('fill-opacity', isPlayer ? '0.92' : '0.62');
                }
            });

            Object.keys(counts).forEach(k => {
                const cEl = document.getElementById(`cnt-${k}`);
                if (cEl) cEl.textContent = counts[k];
            });

            const myTerr = counts[state.kingdom] || 0;
            document.getElementById('hdr-terr-count').textContent = myTerr;
            document.getElementById('banner-terr-mobile').textContent = currentLang === 'ko'
                ? `영토 ${myTerr} / 60`
                : `Land ${myTerr} / 60`;
        }

        function updateUI() {
            const kData = KINGDOMS[state.kingdom];
            const isKo = currentLang === 'ko';
            const badge = document.getElementById('hdr-kingdom');
            badge.textContent = getPlayerDisplayName();
            badge.className = `px-1.5 py-0.5 rounded text-[10px] font-extrabold border shrink-0 ${kData.badgeCls}`;

            document.getElementById('hdr-ruler').textContent = state.rulerObj ? (isKo ? state.rulerObj.ko : state.rulerObj.en) : '';
            document.getElementById('hdr-year').textContent = isKo
                ? `${state.year}년 (${state.turn}/${state.maxTurns}턴)`
                : `AD ${state.year} (T${state.turn}/${state.maxTurns})`;

            ['economy', 'military', 'authority', 'diplomacy'].forEach(s => {
                const v = state.stats[s];
                document.getElementById(`val-${s}`).textContent = v;
                document.getElementById(`bar-${s}`).style.width = `${v}%`;
            });

            updateMapColors();
        }

        /* =========================================================================
           8. 이벤트 로드 및 간결한 행동 중심 선택지 렌더링
           ========================================================================= */
        function calculateChoiceProb(ch) {
            const avgStat = (state.stats.economy + state.stats.military + state.stats.authority + state.stats.diplomacy) / 4;
            const bonus = Math.round((avgStat - 60) * 0.35);
            return Math.max(25, Math.min(96, (ch.baseProb || 75) + bonus));
        }

        function loadNextTurnEvent() {
            if (state.gameOver) return;

            // 1) 20턴 주기 인류 역사적 변곡점 체크
            if (TURNING_POINT_EVENTS[state.turn]) {
                state.currentEvent = { ...TURNING_POINT_EVENTS[state.turn], isTurningPoint: true };
            }
            // 2) 15~20턴 간격 역사 통계 기반 불가항력 시나리오 (15, 30, 35, 50, 55, 70, 75턴 등)
            else if (state.turn > 1 && (state.turn % 15 === 0 || state.turn % 17 === 0)) {
                const fm = FORCE_MAJEURE_EVENTS[Math.floor(Math.random() * FORCE_MAJEURE_EVENTS.length)];
                state.currentEvent = { ...fm, isForceMajeure: true };
            }
            // 3) 일반 정무 이벤트
            else {
                const ev = GENERAL_EVENTS[Math.floor(Math.random() * GENERAL_EVENTS.length)];
                state.currentEvent = { ...ev };
            }

            state.timeRemaining = 60.0;
            renderChoicesUI();
        }

        function renderChoicesUI() {
            const ev = state.currentEvent;
            if (!ev) return;
            const isKo = currentLang === 'ko';

            document.getElementById('ev-tag').textContent = isKo ? ev.tagKo : ev.tagEn;
            document.getElementById('ev-turn-info').textContent = isKo
                ? `제${state.turn}턴 / ${state.maxTurns}턴 (${state.year}년)`
                : `Turn ${state.turn} / ${state.maxTurns} (AD ${state.year})`;
            document.getElementById('ev-title').textContent = isKo ? ev.titleKo : ev.titleEn;
            document.getElementById('ev-desc').textContent = isKo ? ev.descKo : ev.descEn;

            const box = document.getElementById('choices-box');
            box.innerHTML = '';

            ev.choices.forEach((ch, idx) => {
                const prob = calculateChoiceProb(ch);
                const probColor = prob >= 80 ? 'text-emerald-300 border-emerald-500/40 bg-emerald-500/15'
                    : prob >= 65 ? 'text-amber-300 border-amber-500/40 bg-amber-500/15'
                    : 'text-rose-300 border-rose-500/40 bg-rose-500/15';

                const btn = document.createElement('button');
                btn.className = 'w-full px-3 py-2 rounded-xl bg-[#131c2b] hover:bg-[#1b273b] active:scale-[0.99] border border-white/10 hover:border-amber-400/50 transition flex items-center justify-between gap-2 text-left';
                btn.innerHTML = `
                    <div class="flex items-center gap-2 min-w-0 flex-1">
                        <span class="w-5 h-5 rounded-md bg-amber-500/20 border border-amber-400/40 text-amber-300 text-[10px] font-black flex items-center justify-center shrink-0">${idx + 1}</span>
                        <span class="text-xs font-bold text-gray-100 leading-snug break-words">${isKo ? ch.actKo : ch.actEn}</span>
                    </div>
                    <span class="px-2 py-0.5 rounded-md border text-[10px] font-extrabold shrink-0 ${probColor}">
                        ${isKo ? `성공 ${prob}%` : `${prob}%`}
                    </span>
                `;
                btn.onclick = () => executeChoice(idx);
                box.appendChild(btn);
            });
        }

        /* =========================================================================
           9. 선택 실행 및 결과 팝업 상세 표시 + 영토 변동 처리
           ========================================================================= */
        function changeTerritories(gainZone, gainCnt, loseCnt) {
            const gainedNames = [];
            const lostNames = [];

            if (gainCnt > 0) {
                const candidates = PROVINCES_60.filter(p => state.owners[p.id] !== state.kingdom);
                // 우선순위 영역 먼저
                if (gainZone && gainZone.length > 0) {
                    candidates.sort((a, b) => (gainZone.includes(b.id) ? 1 : 0) - (gainZone.includes(a.id) ? 1 : 0));
                }
                for (let i = 0; i < Math.min(gainCnt, candidates.length); i++) {
                    const target = candidates[i];
                    state.owners[target.id] = state.kingdom;
                    gainedNames.push(currentLang === 'ko' ? target.name : target.nameEn);
                }
            }

            if (loseCnt > 0) {
                const myProvs = PROVINCES_60.filter(p => state.owners[p.id] === state.kingdom);
                for (let i = 0; i < Math.min(loseCnt, Math.max(0, myProvs.length - 1)); i++) {
                    const target = myProvs[myProvs.length - 1 - i];
                    state.owners[target.id] = target.owner !== state.kingdom ? target.owner : 'china';
                    lostNames.push(currentLang === 'ko' ? target.name : target.nameEn);
                }
            }

            return { gainedNames, lostNames };
        }

        function executeChoice(choiceIdx, isTimeout = false) {
            if (state.gameOver || state.isPopupOpen) return;
            state.isPopupOpen = true;

            const ev = state.currentEvent;
            const ch = ev.choices[choiceIdx] || ev.choices[0];
            const prob = calculateChoiceProb(ch);
            const roll = Math.random() * 100;
            const isSuccess = !isTimeout && roll <= prob;
            const isKo = currentLang === 'ko';

            const statDiff = { economy: 0, military: 0, authority: 0, diplomacy: 0 };
            Object.keys(ch.eff || {}).forEach(k => {
                statDiff[k] = isSuccess ? ch.eff[k] : Math.round(ch.eff[k] * -0.45) - 4;
            });

            const gainCount = isSuccess ? (ch.gainCnt ?? 2) : 0;
            const loseCount = isSuccess ? 0 : (ch.loseCnt ?? 1);
            const terrRes = changeTerritories(ch.gainZone, gainCount, loseCount);

            // 파생 연쇄 사건
            const chain = CHAIN_EVENTS[Math.floor(Math.random() * CHAIN_EVENTS.length)];
            Object.keys(chain.eff).forEach(k => {
                statDiff[k] = (statDiff[k] || 0) + chain.eff[k];
            });

            // 스탯 반영
            Object.keys(statDiff).forEach(k => {
                state.stats[k] = clamp(state.stats[k] + statDiff[k]);
            });

            // 중요 역사 기록 저장 (변곡점, 불가항력, 대확장)
            if (ev.isTurningPoint || ev.isForceMajeure || terrRes.gainedNames.length >= 3) {
                state.historyMilestones.push({
                    sceneType: ev.isTurningPoint ? 'turning_point' : (ev.isForceMajeure ? 'disaster' : 'conquest'),
                    year: state.year,
                    turn: state.turn,
                    titleKo: ev.titleKo,
                    titleEn: ev.titleEn,
                    descKo: `${ch.actKo} 결단을 통해 국세를 크게 일신했습니다.`,
                    descEn: `Executed "${ch.actEn}" to reshape the destiny of the realm.`
                });
            }

            updateUI();

            // 결과 팝업 채우기
            const hdrBar = document.getElementById('popup-header-bar');
            const badge = document.getElementById('popup-outcome-badge');
            if (isSuccess) {
                hdrBar.className = 'px-3 py-2 rounded-xl flex items-center justify-between border bg-emerald-950/60 border-emerald-500/40';
                badge.className = 'px-2.5 py-0.5 rounded text-xs font-extrabold bg-emerald-500 text-black';
                badge.textContent = isKo ? '작전 성공' : 'SUCCESS';
            } else {
                hdrBar.className = 'px-3 py-2 rounded-xl flex items-center justify-between border bg-rose-950/60 border-rose-500/40';
                badge.className = 'px-2.5 py-0.5 rounded text-xs font-extrabold bg-rose-500 text-white';
                badge.textContent = isTimeout ? (isKo ? '시간 초과 차질' : 'TIMEOUT') : (isKo ? '작전 차질' : 'SETBACK');
            }

            document.getElementById('popup-prob-detail').textContent = isKo ? `성공 확률 ${prob}% 판정` : `Success Prob ${prob}%`;
            document.getElementById('popup-turn-label').textContent = isKo ? `서기 ${state.year}년 (제${state.turn}턴)` : `AD ${state.year} (Turn ${state.turn})`;
            document.getElementById('popup-main-title').textContent = isKo ? ch.actKo : ch.actEn;
            document.getElementById('popup-main-desc').textContent = isSuccess
                ? (isKo ? `군주의 결단으로 [${ch.actKo}] 작전이 성공리에 완수되어 국력과 영토가 확장되었습니다.` : `Your decision to execute [${ch.actEn}] succeeded brilliantly, strengthening the realm.`)
                : (isKo ? `[${ch.actKo}] 추진 과정에서 거센 저항과 변수가 발생하여 일부 손실을 입었습니다.` : `Unexpected resistance hindered [${ch.actEn}], causing temporary losses.`);

            const chipBox = document.getElementById('popup-stat-chips');
            chipBox.innerHTML = '';
            const statNames = { economy: isKo ? '국고' : 'Treasury', military: isKo ? '군사' : 'Military', authority: isKo ? '민심' : 'Support', diplomacy: isKo ? '외교' : 'Diplomacy' };
            Object.keys(statDiff).forEach(k => {
                const v = statDiff[k];
                if (v === 0) return;
                const sp = document.createElement('span');
                sp.className = `px-2 py-0.5 rounded text-[10px] font-bold border ${v > 0 ? 'bg-emerald-500/15 text-emerald-300 border-emerald-500/30' : 'bg-rose-500/15 text-rose-300 border-rose-500/30'}`;
                sp.textContent = `${statNames[k]} ${v > 0 ? '+' + v : v}`;
                chipBox.appendChild(sp);
            });

            const terrEl = document.getElementById('popup-terr-change');
            if (terrRes.gainedNames.length > 0) {
                terrEl.textContent = isKo
                    ? `★ 수복 및 점령 지역 (+${terrRes.gainedNames.length}): ${terrRes.gainedNames.join(', ')}`
                    : `★ Captured Regions (+${terrRes.gainedNames.length}): ${terrRes.gainedNames.join(', ')}`;
            } else if (terrRes.lostNames.length > 0) {
                terrEl.textContent = isKo
                    ? `▼ 상실 지역 (-${terrRes.lostNames.length}): ${terrRes.lostNames.join(', ')}`
                    : `▼ Lost Regions (-${terrRes.lostNames.length}): ${terrRes.lostNames.join(', ')}`;
            } else {
                terrEl.textContent = isKo ? '영토 경계 변동 없음 (내실 강화)' : 'No border change (Internal consolidation)';
            }

            document.getElementById('popup-chain-title').textContent = isKo ? chain.koTitle : chain.enTitle;
            document.getElementById('popup-chain-desc').textContent = isKo ? chain.koDesc : chain.enDesc;

            document.getElementById('result-popup-overlay').classList.remove('hidden');
        }

        function closeResultPopupAndAdvance() {
            document.getElementById('result-popup-overlay').classList.add('hidden');
            state.isPopupOpen = false;

            const myTerrCount = Object.values(state.owners).filter(o => o === state.kingdom).length;

            // 1) 동아시아 60분획 전 영토 점령 시 즉시 정복 승리 종료!
            if (myTerrCount >= 60) {
                triggerGameEnding('total_conquest');
                return;
            }

            // 2) 국가 멸망 체크
            if (myTerrCount <= 0 || state.stats.economy <= 0 || state.stats.military <= 0 || state.stats.authority <= 0 || state.stats.diplomacy <= 0) {
                triggerGameEnding('fall');
                return;
            }

            // 3) 영토 30분획 이상 또는 외교 100 도달 시 새로운 국호 선포 모달 (1회)
            if (!state.hasProclaimedEmpire && (myTerrCount >= 30 || state.stats.diplomacy >= 100)) {
                state.hasProclaimedEmpire = true;
                openEmpireProclamationModal();
                return;
            }

            proceedToNextDecadeTurn();
        }

        function proceedToNextDecadeTurn() {
            if (state.turn >= state.maxTurns) {
                triggerGameEnding('completed');
                return;
            }
            // 1턴 = 10년 진행
            state.turn += 1;
            state.year += 10;
            updateUI();
            loadNextTurnEvent();
        }

        /* =========================================================================
           10. 새로운 국호 선포 모달 제어
           ========================================================================= */
        function openEmpireProclamationModal() {
            state.isPopupOpen = true;
            const input = document.getElementById('custom-empire-input');
            if (input) input.value = '';
            document.getElementById('empire-modal-overlay').classList.remove('hidden');
        }

        function selectPresetEmpireName(name) {
            const input = document.getElementById('custom-empire-input');
            if (input) input.value = name;
        }

        function confirmNewEmpireName() {
            const input = document.getElementById('custom-empire-input');
            const val = (input && input.value.trim()) ? input.value.trim() : '대동방 제국';
            state.customEmpireName = val;
            document.getElementById('empire-modal-overlay').classList.add('hidden');
            state.isPopupOpen = false;

            state.historyMilestones.push({
                sceneType: 'empire',
                year: state.year,
                turn: state.turn,
                titleKo: `새로운 대제국 국호 [${val}] 천하 선포`,
                titleEn: `Proclaimed New Empire Title: [${val}]`,
                descKo: `서기 ${state.year}년, 광대한 영토와 외교력을 바탕으로 [${val}]의 탄생을 만방에 알렸습니다.`,
                descEn: `In AD ${state.year}, proclaimed the birth of [${val}] across the world.`
            });

            updateUI();
            proceedToNextDecadeTurn();
        }

        /* =========================================================================
           11. 게임 종료 판정 및 30초 분량 HD 도트 픽셀 역사 요약 슬라이드쇼(GIF 스타일)
           ========================================================================= */
        function triggerGameEnding(endType) {
            state.gameOver = true;
            state.isPopupOpen = false;

            const isKo = currentLang === 'ko';
            const myTerr = Object.values(state.owners).filter(o => o === state.kingdom).length;
            const nationName = getPlayerDisplayName();

            let badgeTxt = '';
            let titleTxt = '';
            let descTxt = '';

            if (endType === 'total_conquest') {
                badgeTxt = isKo ? '★ 동아시아 60분획 완전 제패 · 정복 승리 ★' : '★ ALL 60 REGIONS CONQUERED · CONQUEST VICTORY ★';
                titleTxt = isKo ? `[정복 승리] ${nationName}, 동아시아 천하대일통 달성!` : `[CONQUEST VICTORY] ${nationName} Unifies All of East Asia!`;
                descTxt = isKo
                    ? `서기 ${state.year}년(제${state.turn}턴), 한반도·만주·북방 초원·중원 대륙·왜 열도와 류큐까지 60개 모든 분획을 완벽히 평정하고 인류 역사상 유례없는 초거대 대제국을 완성했습니다!`
                    : `In AD ${state.year} (Turn ${state.turn}), your empire conquered all 60 regions across Korea, Manchuria, China, Steppe, and Japan!`;
            } else if (endType === 'fall') {
                badgeTxt = isKo ? '왕조 멸망 · 역사 속으로 사라지다' : 'DYNASTY FALLEN';
                titleTxt = isKo ? `${nationName}의 비극적 종언` : `Tragic Fall of ${nationName}`;
                descTxt = isKo
                    ? `서기 ${state.year}년, 국력 고갈과 외세의 침공을 이기지 못하고 도성이 함락되었습니다.`
                    : `In AD ${state.year}, the capital fell amidst exhaustion of national strength.`;
            } else {
                if (myTerr >= 35) {
                    badgeTxt = isKo ? '동아시아 패권 대제국 완성' : 'HEGEMONIC GRAND EMPIRE';
                    titleTxt = isKo ? `${nationName}, 천하의 패자로 군림하다` : `${nationName} Reigns Supreme over East Asia`;
                    descTxt = isKo
                        ? `총 ${myTerr}개 분획을 장악하며 중원과 초원, 열도를 모두 아우르는 불멸의 대제국을 건설했습니다.`
                        : `Controlling ${myTerr} regions, you built an immortal superpower across East Asia.`;
                } else {
                    badgeTxt = isKo ? '삼한 수호와 번영의 성군' : 'GUARDIAN MONARCH OF PROSPERITY';
                    titleTxt = isKo ? `${nationName}의 평화와 번영` : `Peace and Prosperity of ${nationName}`;
                    descTxt = isKo
                        ? `격동하는 수백 년의 세월 속에서도 ${myTerr}개 영토와 백성을 굳건히 지켜냈습니다.`
                        : `Across centuries of turmoil, you safeguarded ${myTerr} territories and your people.`;
                }
            }

            state.historyMilestones.push({
                sceneType: endType === 'total_conquest' ? 'victory' : (endType === 'fall' ? 'fall' : 'victory'),
                year: state.year,
                turn: state.turn,
                titleKo: titleTxt,
                titleEn: titleTxt,
                descKo: descTxt,
                descEn: descTxt
            });

            document.getElementById('choices-box').classList.add('hidden');
            document.getElementById('timer-control-bar').classList.add('hidden');

            const endBox = document.getElementById('ending-box');
            endBox.classList.remove('hidden');

            // 30초 분량 디테일 도트 슬라이드쇼 캔버스를 엔딩 박스에 동적 구성
            endBox.innerHTML = `
                <div class="p-3 rounded-2xl bg-[#090f1a] border-2 border-amber-500/50 space-y-2.5 shadow-2xl">
                    <div class="flex items-center justify-between">
                        <span class="px-2 py-0.5 rounded bg-amber-500/20 border border-amber-400/40 text-[10px] font-extrabold text-amber-300">${badgeTxt}</span>
                        <span class="text-[10px] font-bold text-emerald-300">영토 ${myTerr} / 60</span>
                    </div>
                    <h3 class="serif-kr text-sm sm:text-base font-black text-white leading-snug break-words">${titleTxt}</h3>
                    <p class="text-[11px] text-gray-300 leading-relaxed break-words">${descTxt}</p>

                    <!-- 30초 분량 도트 픽셀 역사 요약 슬라이드쇼(GIF 스타일 영상) -->
                    <div class="rounded-xl overflow-hidden border border-white/15 bg-black relative">
                        <div class="px-2.5 py-1 bg-black/80 border-b border-white/10 flex items-center justify-between text-[10px]">
                            <span class="font-extrabold text-amber-300 flex items-center gap-1">
                                <i class="fa-solid fa-film"></i>
                                <span>${isKo ? '왕조 역사 요약 도트 시네마 (30초 GIF 슬라이드쇼)' : 'Dynasty Pixel History Cinema (30s GIF Slideshow)'}</span>
                            </span>
                            <span id="slideshow-timer-txt" class="font-bold text-gray-300 tabular-nums">0.0s / 30.0s</span>
                        </div>
                        <canvas id="ending-slideshow-canvas" width="640" height="320" class="w-full h-44 object-cover pixel-canvas block"></canvas>
                        <div class="p-2 bg-[#0b1220] border-t border-white/10">
                            <div class="flex items-center justify-between text-[10px] text-amber-300 font-bold mb-0.5">
                                <span id="slide-era-label">제1장 · 건국과 출정</span>
                                <span id="slide-step-indicator">1 / 6</span>
                            </div>
                            <p id="slide-caption-text" class="text-[11px] text-white font-semibold leading-snug break-words"></p>
                        </div>
                        <div class="w-full h-1 bg-gray-800">
                            <div id="slideshow-bar" class="h-full bg-amber-400 transition-all duration-150" style="width:0%"></div>
                        </div>
                    </div>

                    <button onclick="initNewRandomGame()" class="w-full py-3 rounded-xl bg-gradient-to-r from-amber-500 to-orange-500 hover:from-amber-400 hover:to-orange-400 text-black font-black text-xs transition shadow-lg">
                        ${isKo ? '새로운 무작위 왕조로 다시 시작하기' : 'Play Again with New Random Dynasty'}
                    </button>
                </div>
            `;

            start30SecEndingSlideshow();
        }

        function build6SlidesFromHistory() {
            const m = state.historyMilestones;
            const myTerr = Object.values(state.owners).filter(o => o === state.kingdom).length;
            const nation = getPlayerDisplayName();
            const isKo = currentLang === 'ko';

            const slides = [
                m[0] || { sceneType: 'founding', year: state.year, titleKo: `${nation}의 건국`, descKo: '위대한 여정의 시작' },
                m[1] || { sceneType: 'conquest', year: state.year - 30, titleKo: isKo ? '국경 확장과 철기병 출정' : 'Border Expansion & Cavalry March', descKo: isKo ? '주변 요충지를 평정하며 세력을 넓혔습니다.' : 'Expanded territory across key strategic basins.' },
                m.find(x => x.sceneType === 'turning_point') || { sceneType: 'turning_point', year: state.year - 20, titleKo: isKo ? '인류 역사적 변곡점 채택' : 'Historical Turning Point Adopted', descKo: isKo ? '화약·금속활자·신사상을 받아들여 문명을 혁신했습니다.' : 'Adopted revolutionary technology and statecraft.' },
                m.find(x => x.sceneType === 'disaster') || { sceneType: 'disaster', year: state.year - 15, titleKo: isKo ? '시대적 대재앙과 국난 극복' : 'Overcoming Great Historical Crises', descKo: isKo ? '기후 변화와 외세의 침입을 군신 일체로 극복했습니다.' : 'United the nation to overcome natural and military crises.' },
                m.find(x => x.sceneType === 'empire') || { sceneType: 'empire', year: state.year - 10, titleKo: isKo ? `${nation}의 천하 위상 확립` : `Rise of ${nation}`, descKo: isKo ? `동아시아 ${myTerr}개 구역에 강력한 영향력을 행사했습니다.` : `Projected hegemony across ${myTerr} regions of East Asia.` },
                m[m.length - 1]
            ];
            return slides;
        }

        function stopSlideshowPlayer() {
            if (slideshowInterval) {
                clearInterval(slideshowInterval);
                slideshowInterval = null;
            }
        }

        function start30SecEndingSlideshow() {
            stopSlideshowPlayer();
            const slides = build6SlidesFromHistory();
            slideshowStartTime = performance.now();
            let animFrame = 0;

            slideshowInterval = setInterval(() => {
                const canvas = document.getElementById('ending-slideshow-canvas');
                if (!canvas) {
                    stopSlideshowPlayer();
                    return;
                }
                animFrame++;
                const elapsedSec = ((performance.now() - slideshowStartTime) / 1000) % 30;
                const slideIdx = Math.min(5, Math.floor(elapsedSec / 5)); // 6개 슬라이드 * 5초 = 30초
                const curSlide = slides[slideIdx];
                const isKo = currentLang === 'ko';

                const timerTxt = document.getElementById('slideshow-timer-txt');
                if (timerTxt) timerTxt.textContent = `${elapsedSec.toFixed(1)}s / 30.0s`;

                const bar = document.getElementById('slideshow-bar');
                if (bar) bar.style.width = `${(elapsedSec / 30) * 100}%`;

                const eraLbl = document.getElementById('slide-era-label');
                if (eraLbl) eraLbl.textContent = isKo
                    ? `제${slideIdx + 1}장 (${curSlide.year}년) · ${curSlide.titleKo}`
                    : `Chapter ${slideIdx + 1} (AD ${curSlide.year}) · ${curSlide.titleEn || curSlide.titleKo}`;

                const stepInd = document.getElementById('slide-step-indicator');
                if (stepInd) stepInd.textContent = `${slideIdx + 1} / 6`;

                const capTxt = document.getElementById('slide-caption-text');
                if (capTxt) capTxt.textContent = isKo ? curSlide.descKo : (curSlide.descEn || curSlide.descKo);

                drawSlideshowPixelFrame(canvas, curSlide.sceneType, slideIdx, animFrame);
            }, 60);
        }

        function drawSlideshowPixelFrame(canvas, sceneType, slideIdx, tick) {
            const ctx = canvas.getContext('2d');
            const W = canvas.width;
            const H = canvas.height;

            const palettes = [
                { top: '#0f172a', bot: '#1e293b', ground: '#1c1917' },
                { top: '#2a0808', bot: '#7f1d1d', ground: '#292524' },
                { top: '#1e1b4b', bot: '#312e81', ground: '#1e293b' },
                { top: '#042f2e', bot: '#115e59', ground: '#134e4a' },
                { top: '#3b0764', bot: '#7e22ce', ground: '#2e1065' },
                { top: '#451a03', bot: '#d97706', ground: '#27171a' }
            ];
            const pal = palettes[slideIdx % palettes.length];

            const grad = ctx.createLinearGradient(0, 0, 0, H);
            grad.addColorStop(0, pal.top);
            grad.addColorStop(1, pal.bot);
            ctx.fillStyle = grad;
            ctx.fillRect(0, 0, W, H);

            // 태양 또는 달
            ctx.fillStyle = slideIdx === 5 ? '#fde047' : '#f59e0b';
            ctx.fillRect(W - 110, 36, 44, 44);

            // 원경 산맥
            ctx.fillStyle = 'rgba(15, 23, 42, 0.75)';
            for (let x = 0; x < W; x += 40) {
                const mh = 50 + ((x * 13 + slideIdx * 29) % 60);
                ctx.fillRect(x, H - 70 - mh, 44, mh);
            }

            // 성벽 및 왕궁
            ctx.fillStyle = '#334155';
            ctx.fillRect(0, H - 90, W, 30);
            ctx.fillStyle = '#991b1b';
            ctx.fillRect(W / 2 - 80, H - 145, 160, 55);
            ctx.fillStyle = '#fbbf24';
            ctx.fillRect(W / 2 - 92, H - 155, 184, 14);

            // 지면
            ctx.fillStyle = pal.ground;
            ctx.fillRect(0, H - 60, W, 60);

            // 행군하는 도트 기병과 군사들 (애니메이션)
            const kColor = FACTION_META[state.kingdom].fill;
            for (let i = 0; i < 10; i++) {
                const sx = ((i * 68) + tick * 4) % (W + 40) - 20;
                const sy = H - 52 + (i % 2) * 14;
                // 말 몸통
                ctx.fillStyle = '#78350f';
                ctx.fillRect(sx, sy, 22, 10);
                // 기수 갑옷
                ctx.fillStyle = kColor;
                ctx.fillRect(sx + 5, sy - 12, 10, 12);
                // 투구와 창
                ctx.fillStyle = '#fde047';
                ctx.fillRect(sx + 6, sy - 16, 8, 4);
                ctx.fillStyle = '#e2e8f0';
                ctx.fillRect(sx + 18, sy - 24, 3, 24);
            }

            // 승리 폭죽/별빛 효과
            if (slideIdx >= 4) {
                for (let s = 0; s < 16; s++) {
                    const px = (s * 43 + tick * 3) % W;
                    const py = (s * 29 + tick * 2) % (H - 120);
                    ctx.fillStyle = s % 2 === 0 ? '#fde047' : '#34d399';
                    ctx.fillRect(px, py, 4, 4);
                }
            }
        }

        /* =========================================================================
           12. 2배 고화질(HD) 실시간 도트 배너(680x112) 및 인트로(640x360) 루프
           ========================================================================= */
        function render2xHDDotBanner() {
            const canvas = document.getElementById('dot-banner');
            if (!canvas) return;
            const ctx = canvas.getContext('2d');
            const W = canvas.width;
            const H = canvas.height;

            const avg = (state.stats.economy + state.stats.military + state.stats.authority + state.stats.diplomacy) / 4;
            const skyGrad = ctx.createLinearGradient(0, 0, 0, H);
            if (avg >= 70) {
                skyGrad.addColorStop(0, '#0f172a');
                skyGrad.addColorStop(1, '#b45309');
            } else if (avg <= 35) {
                skyGrad.addColorStop(0, '#2a0808');
                skyGrad.addColorStop(1, '#7f1d1d');
            } else {
                skyGrad.addColorStop(0, '#091526');
                skyGrad.addColorStop(1, '#1e3a5f');
            }
            ctx.fillStyle = skyGrad;
            ctx.fillRect(0, 0, W, H);

            // 능선
            ctx.fillStyle = '#0f172a';
            for (let x = 0; x < W; x += 32) {
                const mh = 24 + ((x * 7) % 28);
                ctx.fillRect(x, H - 28 - mh, 34, mh);
            }

            // 왕궁 및 성곽 (2x HD 디테일)
            ctx.fillStyle = '#334155';
            ctx.fillRect(0, H - 34, W, 12);
            ctx.fillStyle = FACTION_META[state.kingdom].fill;
            ctx.fillRect(W / 2 - 48, H - 68, 96, 34);
            ctx.fillStyle = '#fbbf24';
            ctx.fillRect(W / 2 - 56, H - 74, 112, 8);

            // 지면
            ctx.fillStyle = '#1c1917';
            ctx.fillRect(0, H - 22, W, 22);

            // 움직이는 병사 도트
            const unitCount = Math.max(3, Math.floor(state.stats.military / 10));
            for (let i = 0; i < unitCount; i++) {
                const ux = ((i * 64) + bannerTick * 2) % W;
                ctx.fillStyle = FACTION_META[state.kingdom].fill;
                ctx.fillRect(ux, H - 28, 8, 12);
                ctx.fillStyle = '#e2e8f0';
                ctx.fillRect(ux + 6, H - 36, 2, 16);
            }
        }

        function render2xHDIntroCanvas() {
            if (!isIntroActive) return;
            const canvas = document.getElementById('intro-pixel-canvas');
            if (!canvas) return;
            const ctx = canvas.getContext('2d');
            const W = canvas.width;
            const H = canvas.height;
            const sc = INTRO_PIXEL_SCENES[selectedIntroSceneIndex] || INTRO_PIXEL_SCENES[0];

            const grad = ctx.createLinearGradient(0, 0, 0, H);
            grad.addColorStop(0, sc.skyTop);
            grad.addColorStop(1, sc.skyBot);
            ctx.fillStyle = grad;
            ctx.fillRect(0, 0, W, H);

            // 달/태양
            ctx.fillStyle = '#fde047';
            ctx.fillRect(W - 140, 48, 56, 56);

            // 산맥과 성채
            ctx.fillStyle = 'rgba(10, 15, 28, 0.8)';
            for (let x = 0; x < W; x += 48) {
                const h = 70 + ((x * 19) % 80);
                ctx.fillRect(x, H - 90 - h, 52, h);
            }

            ctx.fillStyle = sc.ground;
            ctx.fillRect(0, H - 90, W, 90);

            // 도트 기병 행렬
            for (let i = 0; i < 8; i++) {
                const cx = ((i * 88) + bannerTick * 3) % (W + 60) - 30;
                const cy = H - 72 + (i % 2) * 18;
                ctx.fillStyle = '#78350f';
                ctx.fillRect(cx, cy, 28, 14);
                ctx.fillStyle = sc.accent;
                ctx.fillRect(cx + 6, cy - 16, 12, 16);
                ctx.fillStyle = '#f8fafc';
                ctx.fillRect(cx + 22, cy - 28, 3, 28);
            }
        }

        function mainLoop(now) {
            const dt = (now - state.lastFrameTime) / 1000;
            state.lastFrameTime = now;
            bannerTick++;

            if (isIntroActive) {
                render2xHDIntroCanvas();
            } else {
                render2xHDDotBanner();
                if (!state.gameOver && !state.isPopupOpen) {
                    state.timeRemaining -= dt * state.speedMultiplier;
                    if (state.timeRemaining <= 0) {
                        state.timeRemaining = 0;
                        executeChoice(0, true);
                    }
                    const secEl = document.getElementById('timer-sec-text');
                    if (secEl) {
                        secEl.textContent = currentLang === 'ko'
                            ? `${state.timeRemaining.toFixed(1)}초`
                            : `${state.timeRemaining.toFixed(1)}s`;
                    }
                    const barEl = document.getElementById('timer-progress-bar');
                    if (barEl) {
                        barEl.style.width = `${(state.timeRemaining / 60) * 100}%`;
                    }
                }
            }
            requestAnimationFrame(mainLoop);
        }

        window.addEventListener('DOMContentLoaded', () => {
            initVisitorCounter();
            pickRandomIntroScene();
            initNewRandomGame();
            requestAnimationFrame(mainLoop);
        });
    </script>
</body>
</html>
