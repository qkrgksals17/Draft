# Draft
import React, { useState, useEffect, useRef } from 'react';
import { Crown, User, Shield, Zap, Crosshair, Star, Share2, RefreshCw } from 'lucide-react';

// --- 데이터 정의 ---

const POSITIONS = {
  TOP: 'TOP',
  JUNGLE: 'JUNGLE',
  MID: 'MID',
  ADC: 'ADC',
  SUPPORT: 'SUPPORT'
};

const TIERS = {
  CHALLENGER: { name: 'Challenger', value: 100, color: 'text-yellow-400' },
  GRANDMASTER: { name: 'GrandMaster', value: 90, color: 'text-red-500' },
  MASTER: { name: 'Master', value: 80, color: 'text-purple-500' },
  DIAMOND: { name: 'Diamond', value: 70, color: 'text-blue-400' },
  EMERALD: { name: 'Emerald', value: 60, color: 'text-green-400' },
  PLATINUM: { name: 'Platinum', value: 50, color: 'text-teal-400' },
  GOLD: { name: 'Gold', value: 40, color: 'text-yellow-600' },
  SILVER: { name: 'Silver', value: 30, color: 'text-gray-400' },
};

// 선수 데이터 (요청사항 반영)
const PLAYERS_DATA = [
  // TOP
  { id: 't1', name: '운타라', position: POSITIONS.TOP, tier: TIERS.CHALLENGER, desc: '전프로' },
  { id: 't2', name: '김뿡', position: POSITIONS.TOP, tier: TIERS.DIAMOND, desc: '' },
  { id: 't3', name: '룩삼', position: POSITIONS.TOP, tier: TIERS.PLATINUM, desc: '' },
  { id: 't4', name: '얍얍', position: POSITIONS.TOP, tier: TIERS.PLATINUM, desc: '' },
  { id: 't5', name: '한동숙', position: POSITIONS.TOP, tier: TIERS.GOLD, desc: '' },
  
  // MID
  { id: 'm1', name: '앰비션', position: POSITIONS.MID, tier: TIERS.CHALLENGER, desc: '전프로/월즈위너' },
  { id: 'm2', name: '인섹', position: POSITIONS.MID, tier: TIERS.CHALLENGER, desc: '전프로' },
  { id: 'm3', name: '피닉스박', position: POSITIONS.MID, tier: TIERS.CHALLENGER, desc: '전프로' },
  { id: 'm4', name: '랄로', position: POSITIONS.MID, tier: TIERS.MASTER, desc: '' },
  { id: 'm5', name: '트롤야', position: POSITIONS.MID, tier: TIERS.DIAMOND, desc: '' },

  // ADC
  { id: 'a1', name: '괴물쥐', position: POSITIONS.ADC, tier: TIERS.GRANDMASTER, desc: '' },
  { id: 'a2', name: '러너', position: POSITIONS.ADC, tier: TIERS.PLATINUM, desc: '골드~플레' },
  { id: 'a3', name: '따효니', position: POSITIONS.ADC, tier: TIERS.PLATINUM, desc: '골드~플레' },
  { id: 'a4', name: '삼식', position: POSITIONS.ADC, tier: TIERS.PLATINUM, desc: '골드~플레' },
  { id: 'a5', name: '명예훈장', position: POSITIONS.ADC, tier: TIERS.PLATINUM, desc: '골드~플레' },

  // SUPPORT
  { id: 's1', name: '순당무', position: POSITIONS.SUPPORT, tier: TIERS.CHALLENGER, desc: '' },
  { id: 's2', name: '인간젤리', position: POSITIONS.SUPPORT, tier: TIERS.CHALLENGER, desc: '전프로' },
  { id: 's3', name: '던', position: POSITIONS.SUPPORT, tier: TIERS.GOLD, desc: '' },
  { id: 's4', name: '소풍왔니', position: POSITIONS.SUPPORT, tier: TIERS.GOLD, desc: '' },
  { id: 's5', name: '푸린', position: POSITIONS.SUPPORT, tier: TIERS.GOLD, desc: '' },
];

// 팀장 (JUNGLE)
const CAPTAINS = [
  { id: 'c1', name: '갱맘', tier: TIERS.CHALLENGER, desc: '전프로' },
  { id: 'c2', name: '뱅', tier: TIERS.CHALLENGER, desc: '전프로(정글 첫출전)' },
  { id: 'c3', name: '소우릎', tier: TIERS.CHALLENGER, desc: '전프로' },
  { id: 'c4', name: '울프', tier: TIERS.CHALLENGER, desc: '전프로(정글 첫출전)' },
  { id: 'c5', name: '큐베', tier: TIERS.CHALLENGER, desc: '전프로' },
];

// --- 컴포넌트 ---

export default function DraftSimulator() {
  const [step, setStep] = useState('INTRO'); // INTRO, ORDER, DRAFT, RESULT
  const [userCaptainId, setUserCaptainId] = useState(null);
  const [teams, setTeams] = useState({}); // { captainId: { captain: {}, members: [] } }
  const [draftOrder, setDraftOrder] = useState([]); // Array of captainIds
  const [currentPickIndex, setCurrentPickIndex] = useState(0);
  const [availablePlayers, setAvailablePlayers] = useState(PLAYERS_DATA);
  const [logs, setLogs] = useState([]);
  
  const logBoxRef = useRef(null);

  // 초기화
  const initializeGame = (selectedCaptainId) => {
    setUserCaptainId(selectedCaptainId);
    
    // 팀 초기화
    const initialTeams = {};
    CAPTAINS.forEach(cap => {
      initialTeams[cap.id] = {
        captain: cap,
        members: [] // { ...player }
      };
    });
    setTeams(initialTeams);
    setAvailablePlayers(PLAYERS_DATA);
    setLogs([{ text: "드래프트에 오신 것을 환영합니다! 팀장을 선택했습니다.", type: 'info' }]);
    setStep('ORDER');
  };

  // 순서 정하기 (랜덤)
  const randomizeOrder = () => {
    const shuffled = [...CAPTAINS].sort(() => Math.random() - 0.5).map(c => c.id);
    setDraftOrder(shuffled);
    setLogs(prev => [...prev, { text: "경매 순서가 랜덤으로 결정되었습니다!", type: 'system' }]);
    
    // 스네이크 드래프트 순서 생성 (4라운드)
    // 라운드 1: 0, 1, 2, 3, 4
    // 라운드 2: 4, 3, 2, 1, 0
    // 라운드 3: 0, 1, 2, 3, 4
    // 라운드 4: 4, 3, 2, 1, 0
    
    setTimeout(() => setStep('DRAFT'), 1500);
  };

  // 현재 턴 계산 로직 (스네이크)
  const getCurrentTurnCaptainId = () => {
    const round = Math.floor(currentPickIndex / 5);
    const indexInRound = currentPickIndex % 5;
    
    // 짝수 라운드(0, 2)는 정방향, 홀수 라운드(1, 3)는 역방향
    const actualIndex = (round % 2 === 0) ? indexInRound : 4 - indexInRound;
    return draftOrder[actualIndex];
  };

  // AI 자동 픽 로직
  useEffect(() => {
    if (step !== 'DRAFT') return;
    if (currentPickIndex >= 20) {
      setStep('RESULT');
      return;
    }

    const currentCaptainId = getCurrentTurnCaptainId();
    
    // 내 턴이면 대기
    if (currentCaptainId === userCaptainId) return;

    // AI 턴이면 1초 후 픽
    const timer = setTimeout(() => {
      aiPick(currentCaptainId);
    }, 1000);

    return () => clearTimeout(timer);
  }, [step, currentPickIndex, availablePlayers]);

  // 로그 스크롤
  useEffect(() => {
    if (logBoxRef.current) {
      logBoxRef.current.scrollTop = logBoxRef.current.scrollHeight;
    }
  }, [logs]);

  const aiPick = (captainId) => {
    const currentTeam = teams[captainId];
    const filledPositions = currentTeam.members.map(m => m.position);
    
    // 필요한 포지션 찾기
    const neededPositions = Object.values(POSITIONS).filter(p => p !== POSITIONS.JUNGLE && !filledPositions.includes(p));
    
    // 남은 선수 중 필요한 포지션의 선수 필터링
    const candidates = availablePlayers.filter(p => neededPositions.includes(p.position));
    
    // 티어 순으로 정렬 (높은 티어 우선)
    candidates.sort((a, b) => b.tier.value - a.tier.value);

    if (candidates.length > 0) {
      const pick = candidates[0];
      confirmPick(captainId, pick);
    } else {
      console.error("AI Pick Error: No candidates found");
    }
  };

  const userPick = (player) => {
    const currentCaptainId = getCurrentTurnCaptainId();
    if (currentCaptainId !== userCaptainId) return;

    const currentTeam = teams[userCaptainId];
    if (currentTeam.members.some(m => m.position === player.position)) {
      alert("이미 해당 포지션의 선수가 있습니다!");
      return;
    }

    confirmPick(userCaptainId, player);
  };

  const confirmPick = (captainId, player) => {
    setTeams(prev => ({
      ...prev,
      [captainId]: {
        ...prev[captainId],
        members: [...prev[captainId].members, player]
      }
    }));

    setAvailablePlayers(prev => prev.filter(p => p.id !== player.id));
    setLogs(prev => [...prev, { 
      text: `${teams[captainId].captain.name} 팀장이 [${player.position}] ${player.name} (${player.tier.name}) 선수를 지명했습니다!`, 
      type: captainId === userCaptainId ? 'user' : 'enemy' 
    }]);
    setCurrentPickIndex(prev => prev + 1);
  };

  const copyLink = () => {
    const url = window.location.href;
    navigator.clipboard.writeText(url);
    alert("주소가 복사되었습니다. 친구들에게 공유하세요!");
  };

  // --- 렌더링 헬퍼 ---

  const getPositionIcon = (pos) => {
    switch (pos) {
      case POSITIONS.TOP: return <Shield className="w-4 h-4" />;
      case POSITIONS.JUNGLE: return <Crown className="w-4 h-4" />;
      case POSITIONS.MID: return <Zap className="w-4 h-4" />;
      case POSITIONS.ADC: return <Crosshair className="w-4 h-4" />;
      case POSITIONS.SUPPORT: return <User className="w-4 h-4" />;
      default: return null;
    }
  };

  // --- 메인 화면들 ---

  if (step === 'INTRO') {
    return (
      <div className="min-h-screen bg-gray-900 text-white flex flex-col items-center justify-center p-4 font-sans">
        <div className="max-w-4xl w-full text-center space-y-8">
          <div className="space-y-2">
            <h1 className="text-5xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-green-400 to-emerald-600 filter drop-shadow-lg">
              CHZZK CUP DRAFT
            </h1>
            <p className="text-gray-400 text-lg">2025 시즌 자낳대 시뮬레이터</p>
          </div>

          <div className="bg-gray-800/50 p-8 rounded-2xl border border-gray-700 backdrop-blur-sm">
            <h2 className="text-2xl font-bold mb-6 text-emerald-400">당신의 정글러(팀장)를 선택하세요</h2>
            <div className="grid grid-cols-1 sm:grid-cols-3 md:grid-cols-5 gap-4">
              {CAPTAINS.map((cap) => (
                <button
                  key={cap.id}
                  onClick={() => initializeGame(cap.id)}
                  className="group relative flex flex-col items-center p-4 bg-gray-800 border-2 border-gray-700 hover:border-emerald-500 rounded-xl transition-all duration-300 hover:shadow-[0_0_20px_rgba(16,185,129,0.3)]"
                >
                  <div className="w-20 h-20 bg-gray-700 rounded-full flex items-center justify-center mb-3 group-hover:scale-110 transition-transform">
                    <Crown className="w-10 h-10 text-emerald-500" />
                  </div>
                  <span className="text-xl font-bold">{cap.name}</span>
                  <span className="text-xs text-gray-400 mt-1">{cap.desc}</span>
                  <span className={`text-xs font-bold mt-1 ${cap.tier.color}`}>{cap.tier.name}</span>
                </button>
              ))}
            </div>
          </div>
          
          <div className="text-sm text-gray-500">
             * 본 시뮬레이터는 1인용이며, 다른 팀장은 AI가 자동으로 플레이합니다.
          </div>
        </div>
      </div>
    );
  }

  if (step === 'ORDER') {
    return (
      <div className="min-h-screen bg-gray-900 text-white flex flex-col items-center justify-center p-4">
        <div className="text-center space-y-6">
          <h2 className="text-3xl font-bold animate-pulse text-emerald-400">경매 순서 추첨 중...</h2>
          <div className="flex gap-4 justify-center">
             <RefreshCw className="w-12 h-12 animate-spin text-gray-500" />
          </div>
          <button 
            onClick={randomizeOrder}
            className="px-8 py-3 bg-emerald-600 hover:bg-emerald-500 rounded-full font-bold text-lg shadow-lg transition-colors"
          >
            추첨 시작
          </button>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-900 text-gray-100 flex flex-col font-sans">
      {/* Header */}
      <header className="bg-gray-800 border-b border-gray-700 p-4 flex justify-between items-center shadow-lg sticky top-0 z-20">
        <div className="flex items-center gap-3">
          <div className="bg-emerald-500 p-2 rounded-lg">
             <Crown className="w-6 h-6 text-white" />
          </div>
          <div>
            <h1 className="text-xl font-bold text-emerald-400">CHZZK DRAFT</h1>
            <p className="text-xs text-gray-400">
              {step === 'DRAFT' ? `Round ${Math.floor(currentPickIndex / 5) + 1} / 4` : '드래프트 완료'}
            </p>
          </div>
        </div>
        
        {step === 'DRAFT' && (
          <div className="flex-1 max-w-2xl mx-8 hidden md:block">
            <div className="flex justify-between items-center text-xs text-gray-400 mb-1 px-1">
              <span>NEXT PICK</span>
              <span>SNAKE ORDER</span>
            </div>
            <div className="flex gap-1 overflow-hidden">
              {Array.from({length: 5}).map((_, idx) => {
                 // 현재 픽 인덱스부터 5개 보여주기
                 const targetIndex = currentPickIndex + idx;
                 if (targetIndex >= 20) return null;
                 
                 const round = Math.floor(targetIndex / 5);
                 const indexInRound = targetIndex % 5;
                 const actualIndex = (round % 2 === 0) ? indexInRound : 4 - indexInRound;
                 const capId = draftOrder[actualIndex];
                 const isCurrent = idx === 0;
                 
                 return (
                   <div key={targetIndex} 
                     className={`flex-1 h-12 flex items-center justify-center rounded border ${
                       isCurrent 
                         ? 'bg-emerald-900/50 border-emerald-500 text-emerald-300 font-bold' 
                         : 'bg-gray-800 border-gray-700 text-gray-500'
                     }`}
                   >
                     {teams[capId]?.captain.name}
                   </div>
                 );
              })}
            </div>
          </div>
        )}

        <div className="flex items-center gap-4">
          <div className="text-right hidden sm:block">
            <p className="text-sm text-gray-300">내 팀장</p>
            <p className="font-bold text-emerald-400">{teams[userCaptainId]?.captain.name}</p>
          </div>
          {step === 'RESULT' && (
            <button onClick={copyLink} className="p-2 bg-gray-700 hover:bg-gray-600 rounded-full">
              <Share2 className="w-5 h-5" />
            </button>
          )}
        </div>
      </header>

      {/* Main Content */}
      <main className="flex-1 flex flex-col md:flex-row overflow-hidden">
        
        {/* Left: Teams Status */}
        <div className="w-full md:w-1/4 border-r border-gray-700 bg-gray-800/50 p-4 overflow-y-auto">
          <h3 className="text-lg font-bold mb-4 flex items-center gap-2">
            <Shield className="w-5 h-5 text-emerald-500" /> 팀 현황
          </h3>
          <div className="space-y-4">
            {draftOrder.map((capId, idx) => {
              const team = teams[capId];
              const isMe = capId === userCaptainId;
              const isTurn = step === 'DRAFT' && getCurrentTurnCaptainId() === capId;

              return (
                <div key={capId} className={`p-3 rounded-lg border ${isTurn ? 'border-emerald-500 bg-emerald-900/20' : 'border-gray-700 bg-gray-800'} ${isMe ? 'ring-2 ring-emerald-500/30' : ''}`}>
                  <div className="flex justify-between items-center mb-2">
                    <span className={`font-bold ${isMe ? 'text-emerald-400' : 'text-white'}`}>
                      {idx + 1}. {team.captain.name} 팀
                    </span>
                    {isTurn && <span className="text-xs bg-emerald-600 px-2 py-0.5 rounded text-white animate-pulse">PICKING</span>}
                  </div>
                  <div className="space-y-1">
                    {/* Jungle (Captain) */}
                    <div className="flex items-center text-xs text-gray-400">
                      <Crown className="w-3 h-3 mr-1 text-yellow-500" /> 
                      <span className="w-12">JUG</span> 
                      <span className="text-gray-200">{team.captain.name}</span>
                    </div>
                    {/* Members */}
                    {team.members.map(m => (
                      <div key={m.id} className="flex items-center text-xs text-gray-400">
                        <span className="w-4 flex justify-center mr-1">{getPositionIcon(m.position)}</span>
                        <span className="w-12">{m.position.slice(0,3)}</span>
                        <span className={`flex-1 ${m.tier.color}`}>{m.name}</span>
                      </div>
                    ))}
                    {/* Empty Slots */}
                    {Array.from({length: 4 - team.members.length}).map((_, i) => (
                      <div key={`empty-${i}`} className="flex items-center text-xs text-gray-600">
                         <span className="w-4 mr-1"></span>
                         <span className="w-12">---</span>
                         <span>(Empty)</span>
                      </div>
                    ))}
                  </div>
                </div>
              );
            })}
          </div>
        </div>

        {/* Center: Drafting Board (Only visible during Draft) */}
        {step === 'DRAFT' && (
          <div className="flex-1 flex flex-col p-4 overflow-hidden">
            <div className="mb-4 flex justify-between items-center">
              <h3 className="text-xl font-bold">영입 가능 선수</h3>
              <div className="text-sm text-gray-400">
                {getCurrentTurnCaptainId() === userCaptainId 
                  ? <span className="text-emerald-400 font-bold text-lg animate-bounce block">당신의 차례입니다!</span>
                  : <span>{teams[getCurrentTurnCaptainId()]?.captain.name} 팀장이 고민중...</span>
                }
              </div>
            </div>

            <div className="flex-1 overflow-y-auto grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-3 content-start pb-20">
              {availablePlayers.map(player => {
                const myTeam = teams[userCaptainId];
                const isPositionFilled = myTeam.members.some(m => m.position === player.position);
                const isMyTurn = getCurrentTurnCaptainId() === userCaptainId;
                const canPick = isMyTurn && !isPositionFilled;

                return (
                  <button
                    key={player.id}
                    disabled={!canPick}
                    onClick={() => userPick(player)}
                    className={`
                      relative p-4 rounded-xl border text-left transition-all
                      ${canPick 
                        ? 'bg-gray-800 border-gray-600 hover:border-emerald-500 hover:bg-gray-700 hover:scale-105 shadow-lg cursor-pointer' 
                        : 'bg-gray-800/50 border-gray-700 opacity-60 cursor-not-allowed'}
                    `}
                  >
                    <div className="flex justify-between items-start mb-2">
                      <div className="flex items-center gap-2">
                         <div className={`p-1.5 rounded bg-gray-700 text-gray-300`}>
                           {getPositionIcon(player.position)}
                         </div>
                         <div>
                           <div className="font-bold text-lg leading-none">{player.name}</div>
                           <div className="text-xs text-gray-500 mt-0.5">{player.position}</div>
                         </div>
                      </div>
                      <div className={`text-xs font-bold px-2 py-1 rounded bg-gray-900 ${player.tier.color}`}>
                        {player.tier.name}
                      </div>
                    </div>
                    <div className="text-xs text-gray-400 h-4">
                      {player.desc}
                    </div>
                    {canPick && (
                      <div className="absolute inset-0 bg-emerald-500/10 rounded-xl flex items-center justify-center opacity-0 hover:opacity-100 transition-opacity">
                        <span className="bg-emerald-600 text-white px-3 py-1 rounded-full font-bold text-sm shadow-lg">PICK</span>
                      </div>
                    )}
                  </button>
                );
              })}
            </div>
          </div>
        )}

        {/* Result Screen Content */}
        {step === 'RESULT' && (
           <div className="flex-1 p-8 flex flex-col items-center justify-center overflow-y-auto">
              <div className="text-center mb-8">
                <h2 className="text-4xl font-bold text-white mb-2">드래프트 종료!</h2>
                <p className="text-gray-400">완성된 팀 로스터를 확인하세요.</p>
              </div>
              
              <div className="grid grid-cols-1 lg:grid-cols-5 gap-4 w-full max-w-6xl">
                 {draftOrder.map((capId, i) => { // 순위 순서가 아닌 드래프트 순서대로 보여주거나, 원하면 변경 가능
                    const team = teams[capId];
                    const isMe = capId === userCaptainId;
                    return (
                      <div key={capId} className={`bg-gray-800 rounded-xl p-4 border ${isMe ? 'border-emerald-500 shadow-[0_0_15px_rgba(16,185,129,0.2)]' : 'border-gray-700'}`}>
                         <div className="text-center mb-4 border-b border-gray-700 pb-2">
                            <span className="text-xs text-gray-500">Team</span>
                            <div className="font-bold text-xl">{team.captain.name}</div>
                         </div>
                         <div className="space-y-3">
                            <div className="flex justify-between items-center bg-gray-900/50 p-2 rounded">
                               <span className="text-xs text-gray-500 w-8">JUG</span>
                               <span className="font-bold text-sm">{team.captain.name}</span>
                               <span className={`text-xs ${team.captain.tier.color}`}>●</span>
                            </div>
                            {team.members.sort((a,b) => {
                               const order = ['TOP', 'MID', 'ADC', 'SUPPORT'];
                               return order.indexOf(a.position) - order.indexOf(b.position);
                            }).map(m => (
                              <div key={m.id} className="flex justify-between items-center bg-gray-700/30 p-2 rounded">
                                 <span className="text-xs text-gray-500 w-8">{m.position.slice(0,3)}</span>
                                 <span className="font-bold text-sm">{m.name}</span>
                                 <span className={`text-xs ${m.tier.color}`}>●</span>
                              </div>
                            ))}
                         </div>
                      </div>
                    );
                 })}
              </div>
              
              <button onClick={() => setStep('INTRO')} className="mt-12 px-8 py-3 bg-gray-700 hover:bg-gray-600 rounded-full font-bold text-white transition-colors flex items-center gap-2">
                 <RefreshCw className="w-5 h-5" /> 다시 하기
              </button>
           </div>
        )}

        {/* Right: Chat/Logs */}
        <div className="w-full md:w-64 lg:w-80 bg-gray-900 border-l border-gray-800 flex flex-col h-64 md:h-auto shrink-0">
          <div className="p-3 border-b border-gray-800 font-bold text-gray-300 flex items-center gap-2">
            <div className="w-2 h-2 bg-red-500 rounded-full animate-pulse"></div>
            LIVE DRAFT LOG
          </div>
          <div ref={logBoxRef} className="flex-1 overflow-y-auto p-4 space-y-3 font-mono text-sm">
            {logs.map((log, i) => (
              <div key={i} className={`
                p-2 rounded border-l-2 
                ${log.type === 'user' ? 'border-emerald-500 bg-emerald-900/10 text-emerald-100' : 
                  log.type === 'enemy' ? 'border-red-500 bg-red-900/10 text-gray-300' : 
                  'border-gray-500 text-gray-500 italic'}
              `}>
                {log.text}
              </div>
            ))}
          </div>
        </div>

      </main>
    </div>
  );
}
