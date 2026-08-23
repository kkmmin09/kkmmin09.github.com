[Uploading 운빨상점_플레이용.html…]()
---
layout: post
title: "운빨 상점 코드 분석 (HTML/CSS/JS)"
excerpt: "상자를 사서 랜덤 아이템을 얻는 미니 웹게임 '운빨 상점'의 코드를 HTML, CSS, JavaScript로 나눠서 뜯어봤습니다."
comments: true
---

# 운빨 상점 플레이용 링크
[🎮 운빨 상점 플레이하기](/assets/games/lucky-shop.html)

# 운빨 상점 코드 분석

코인으로 상자를 사서 랜덤 아이템을 얻고, 인벤토리에서 팔고, 도감을 채워가는 미니 웹게임입니다. HTML 한 파일 안에 구조(HTML), 스타일(CSS), 동작(JavaScript)이 모두 들어있습니다.

아래에서 부분별로 나눠 설명합니다.

---

## HTML — 구조

```html
<!doctype html>
<html lang="ko">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>운빨 상점</title>
```
문서 언어, 문자 인코딩, 모바일 반응형 뷰포트를 설정합니다.

```html
<header class="top-bar">
  <div class="shop-brand">
    <div class="seal-mark">
      <svg viewBox="0 0 24 24" ...>...</svg>
    </div>
    <div class="shop-titles">
      <h1>운빨 상점</h1>
      <p>행운을 사고파는 곳</p>
    </div>
  </div>
  <div class="stats">
    <div class="stat">
      <span class="stat-label">코인</span>
      <span class="stat-value" id="coin-count">0</span>
    </div>
    ...
    <button id="reset-btn" class="reset-btn">초기화</button>
  </div>
</header>
```
상단 헤더입니다. 로고(인장 아이콘), 게임 제목, 그리고 코인/개봉수/도감 진행도를 보여주는 통계와 초기화 버튼이 있습니다. `id`가 붙은 요소는 JS가 값을 채워 넣는 자리입니다.

```html
<nav class="tab-bar">
  <button class="tab-btn active" data-tab="shop">상점</button>
  <button class="tab-btn" data-tab="inventory">인벤토리</button>
  <button class="tab-btn" data-tab="dex">도감</button>
</nav>
```
탭 메뉴입니다. `data-tab` 속성으로 각 버튼이 어떤 화면을 여는지 표시하고, JS가 클릭을 감지해 화면을 전환합니다.

```html
<main>
  <section id="tab-shop" class="tab-panel active">
    <div class="coin-aid">
      <span class="coin-aid-text">코인이 100 미만일 때만 누를 수 있어요</span>
      <button id="coin-aid-btn" class="coin-aid-btn">5코인 받기</button>
    </div>
    <div id="shop-boxes" class="box-grid"></div>
  </section>

  <section id="tab-inventory" class="tab-panel">
    <div class="inv-toolbar">
      <span class="inv-toolbar-text">아이템을 하나씩 팔지 않고 한 번에 정리할 수 있어요</span>
      <button id="sell-all-inv-btn" class="inv-toolbar-btn">인벤토리 전체 판매</button>
    </div>
    <div id="inventory-list" class="inv-list"></div>
  </section>

  <section id="tab-dex" class="tab-panel">
    <div id="dex-grid" class="dex-grid"></div>
  </section>
</main>
```
상점, 인벤토리, 도감 세 개의 탭 패널입니다. `shop-boxes`, `inventory-list`, `dex-grid`는 빈 컨테이너로 두고, 실제 카드/목록 내용은 전부 JS가 동적으로 채워 넣습니다.

```html
<div id="reveal-modal" class="modal hidden">
  <div id="reveal-card" class="ticket-card">
    <div class="ticket-top">
      <span class="ticket-serial">No. <span id="reveal-serial">000000</span></span>
      <div class="ticket-stamp" id="reveal-stamp"></div>
    </div>
    <div class="ticket-box-row">
      <span id="reveal-box-label"></span>
    </div>
    <div class="ticket-perforation"></div>
    <div class="ticket-body">
      <div class="reveal-new">NEW</div>
      <div class="reveal-name"></div>
      <div class="reveal-rarity"></div>
      <div class="reveal-price"></div>
    </div>
    <div id="reveal-unlock-msg" class="reveal-unlock-msg"></div>
    <button id="reveal-close" class="reveal-close-btn">닫기</button>
  </div>
</div>
```
상자를 개봉했을 때 뜨는 "복권 티켓" 스타일 결과 모달입니다. 평소엔 `hidden` 클래스로 숨겨져 있다가, 개봉 시 JS가 내용을 채우고 보여줍니다.

---

## CSS — 스타일

```css
:root {
  --ground: #f1eee8;
  --panel: #fffdfa;
  --accent: #7a1f2b;
  --gold: #96702a;
  --r-common: #85807a;
  --r-legendary-a: #7a1f2b;
  --r-legendary-b: #c9973f;
  ...
}
```
색상을 변수(디자인 토큰)로 미리 정의해두고, 이후 모든 스타일에서 `var(--accent)` 식으로 재사용합니다. 등급별 색(`--r-common` ~ `--r-legendary`)도 여기서 정의합니다.

```css
@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) { --ground: #17141b; ... }
}
:root[data-theme="dark"] { --ground: #17141b; ... }
```
시스템 설정이 다크모드면 자동으로 색상 변수가 바뀌고, `data-theme="dark"` 속성을 강제로 부여해도 같은 다크 색상이 적용되도록 이중으로 처리했습니다.

```css
.top-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  ...
}
```
상단 헤더의 좌우 배치(로고 vs 통계)를 flexbox로 구성합니다.

```css
.tab-btn.active { color: var(--ink); background: var(--panel-alt); }
.tab-panel { display: none; }
.tab-panel.active { display: block; }
```
활성 탭 버튼은 배경색이 채워지고, 활성화된 탭 패널만 `display: block`으로 보이게 처리합니다. (실제 전환은 JS가 클래스를 토글하며 이루어집니다.)

```css
.box-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(210px, 1fr));
  gap: 20px;
}
```
상점의 상자 카드들을 반응형 그리드로 배치합니다. 화면 너비에 따라 카드 개수가 자동 조정됩니다.

```css
.box-card::before {
  content: "";
  position: absolute;
  top: 0; left: 0; right: 0;
  height: 3px;
  background: var(--tier-color, var(--muted));
}
.box-card[data-box="normal"] { --tier-color: #a5713f; }
.box-card[data-box="rare"] { --tier-color: #9aa3ad; }
.box-card[data-box="legendary"] { --tier-color: var(--gold); }
```
각 상자 카드 위에 등급별 색상 줄을 표시합니다. `data-box` 속성값에 따라 다른 색상 변수를 사용합니다.

```css
.inv-row {
  display: flex;
  align-items: center;
  gap: 14px;
  ...
}
```
인벤토리에서 아이템 한 줄(아이콘+이름+수량+판매버튼)을 가로로 배치합니다.

```css
.dex-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(132px, 1fr));
  gap: 12px;
}
.dex-cell.undiscovered {
  background: var(--panel-alt);
  border-style: dashed;
  opacity: 0.7;
}
```
도감 격자를 구성하고, 아직 발견하지 못한 아이템은 점선 테두리와 흐린 배경으로 구분합니다.

```css
.modal {
  position: fixed;
  inset: 0;
  background: rgba(10, 8, 6, 0.55);
  display: flex;
  align-items: center;
  justify-content: center;
}
.modal.hidden { display: none; }
```
개봉 결과 모달의 배경을 어둡게 깔고 화면 중앙에 카드를 정렬합니다. `hidden` 클래스가 붙으면 화면에서 숨겨집니다.

```css
.ticket-card {
  transform: scale(0.85) translateY(8px);
  opacity: 0;
  transition: transform 0.32s cubic-bezier(0.2, 1.4, 0.4, 1), opacity 0.25s ease;
}
.ticket-card.pop { transform: scale(1) translateY(0); opacity: 1; }
```
결과 티켓이 통통 튀듯 나타나는 애니메이션입니다. `pop` 클래스가 붙는 순간 원래 크기로 커지면서 나타납니다.

```css
.reveal-rarity.is-legendary {
  background: linear-gradient(90deg, var(--r-legendary-a), var(--r-legendary-b));
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
}
```
전설 등급일 때 등급 텍스트에 그라데이션 색을 입히는 효과입니다. 텍스트를 투명하게 하고, 배경 그라데이션이 글자 모양대로만 보이게 클리핑합니다.

---

## JavaScript — 동작

```javascript
const RARITIES = [
  { key: 'common',    label: '일반',   color: 'var(--r-common)', weight: 60.9 },
  { key: 'uncommon',  label: '고급',   color: 'var(--r-uncommon)', weight: 25 },
  { key: 'rare',      label: '희귀',   color: 'var(--r-rare)', weight: 10 },
  { key: 'epic',      label: '영웅',   color: 'var(--r-epic)', weight: 4 },
  { key: 'legendary', label: '전설',   color: 'var(--r-legendary-a)', weight: 0.1 },
];
```
등급별 정보(이름, 색상, 뽑힐 확률 가중치)를 정의합니다. 가중치 합이 100이 되도록 '일반'을 60.9로 보정했습니다.

```javascript
const BOXES = [
  { key: 'normal', label: '일반 상자', cost: 100, icon: 'crate', unlock: { type: 'always' } },
  { key: 'rare', label: '희귀 상자', cost: 300, icon: 'pouch', unlock: { type: 'boxesOpened', value: 10 } },
  { key: 'legendary', label: '전설 상자', cost: 700, icon: 'crown', unlock: { type: 'boxesOpened', value: 30 } },
];
```
상자 종류와 가격, 아이콘, 해금 조건을 정의합니다. 희귀 상자는 10개 개봉 시, 전설 상자는 30개 개봉 시 해금됩니다.

```javascript
const ICONS = {
  crate: '<svg ...>...</svg>',
  pouch: '<svg ...>...</svg>',
  crown: '<svg ...>...</svg>',
  lock: '<svg ...>...</svg>',
};
```
상자/잠금 아이콘을 SVG 문자열로 정의해두고, 필요할 때 HTML에 그대로 삽입합니다.

```javascript
const ITEM_POOL = {
  normal: {
    common: [['나무젓가락', 25], ['낡은 숟가락', 35], ...],
    ...
  },
  rare: { ... },
  legendary: { ... },
};
```
상자별, 등급별로 뽑힐 수 있는 아이템 목록(이름, 판매가)입니다. 같은 등급이라도 좋은 상자일수록 가격이 높게 설정되어 있습니다.

```javascript
const ALL_ITEMS = [];
for (const box of BOXES) {
  const pools = ITEM_POOL[box.key];
  for (const rarity of RARITIES) {
    const list = pools[rarity.key] || [];
    list.forEach(([name, price], idx) => {
      ALL_ITEMS.push({ id: `${box.key}_${rarity.key}_${idx}`, boxKey: box.key, rarityKey: rarity.key, name, price });
    });
  }
}
const ITEM_BY_ID = Object.fromEntries(ALL_ITEMS.map((it) => [it.id, it]));
```
`ITEM_POOL`을 순회하며 전체 아이템 목록(`ALL_ITEMS`)을 만들고, id로 빠르게 찾을 수 있도록 `ITEM_BY_ID` 조회 테이블도 만듭니다.

```javascript
const SAVE_KEY = 'luckyShopSave';
const START_COINS = 1000;

function defaultState() {
  return { coins: START_COINS, boxesOpened: 0, inventory: {}, dex: {} };
}

let state = loadState();

function loadState() {
  try {
    const raw = localStorage.getItem(SAVE_KEY);
    if (!raw) return defaultState();
    const parsed = JSON.parse(raw);
    return { ...defaultState(), ...parsed };
  } catch (e) {
    return defaultState();
  }
}

function saveState() {
  localStorage.setItem(SAVE_KEY, JSON.stringify(state));
}
```
게임 진행 상태(코인, 개봉수, 인벤토리, 도감)를 관리합니다. `localStorage`에 저장해서 새로고침해도 진행 상황이 유지되고, 저장된 값이 깨져 있으면 기본값으로 되돌립니다.

```javascript
function isUnlocked(box) {
  if (box.unlock.type === 'always') return true;
  if (box.unlock.type === 'boxesOpened') return state.boxesOpened >= box.unlock.value;
  return false;
}
```
상자가 해금됐는지 확인합니다. 조건 없이 항상 열려있거나(`always`), 특정 개수 이상 개봉했을 때 해금됩니다.

```javascript
function pickRarity() {
  const total = RARITIES.reduce((s, r) => s + r.weight, 0);
  let roll = Math.random() * total;
  for (const r of RARITIES) {
    if (roll < r.weight) return r;
    roll -= r.weight;
  }
  return RARITIES[0];
}
```
가중치 기반 랜덤 추첨 함수입니다. 전체 가중치 합만큼의 범위에서 난수를 뽑고, 누적 차감하며 어느 등급 구간에 떨어지는지 찾습니다.

```javascript
function pickItem(boxKey, rarityKey) {
  const pool = ITEM_POOL[boxKey][rarityKey];
  const [name, price] = pool[Math.floor(Math.random() * pool.length)];
  return ALL_ITEMS.find((it) => it.boxKey === boxKey && it.rarityKey === rarityKey && it.name === name);
}
```
정해진 상자/등급 안에서 아이템 하나를 랜덤으로 뽑습니다.

```javascript
function buyBox(boxKey) {
  const box = BOXES.find((b) => b.key === boxKey);
  if (!box) return;
  if (!isUnlocked(box)) return;
  if (state.coins < box.cost) return;

  state.coins -= box.cost;
  const rarity = pickRarity();
  const item = pickItem(box.key, rarity.key);

  const isNew = !state.dex[item.id];
  state.dex[item.id] = true;
  state.inventory[item.id] = (state.inventory[item.id] || 0) + 1;
  state.boxesOpened += 1;

  const justUnlocked = BOXES.filter(
    (b) => b.unlock.type === 'boxesOpened' && b.unlock.value === state.boxesOpened
  );

  saveState();
  renderAll();
  showReveal(box, rarity, item, isNew, justUnlocked);
}
```
상자 구매의 핵심 로직입니다. 해금 여부와 코인 잔액을 확인한 뒤 코인을 차감하고, 등급→아이템을 랜덤으로 뽑아 인벤토리와 도감에 반영합니다. 이번 개봉으로 새로 해금된 상자가 있는지도 확인해서 결과창에 함께 보여줍니다.

```javascript
function sellItem(itemId, qty) {
  const owned = state.inventory[itemId] || 0;
  const sellQty = Math.min(qty, owned);
  if (sellQty <= 0) return;
  const item = ITEM_BY_ID[itemId];
  state.coins += item.price * sellQty;
  state.inventory[itemId] -= sellQty;
  if (state.inventory[itemId] <= 0) delete state.inventory[itemId];
  saveState();
  renderAll();
}
```
아이템을 지정한 수량만큼 판매해 코인으로 바꿉니다. 보유 수량을 초과해서 팔 수 없도록 `Math.min`으로 제한합니다.

```javascript
function sellAllInventory() {
  const entries = Object.entries(state.inventory);
  if (entries.length === 0) return;
  const total = entries.reduce((sum, [itemId, qty]) => sum + ITEM_BY_ID[itemId].price * qty, 0);
  if (!confirm(`보유한 아이템 전체를 판매하고 ${total.toLocaleString()} 코인을 받을까요?`)) return;
  state.coins += total;
  state.inventory = {};
  saveState();
  renderAll();
}
```
인벤토리 전체를 한 번에 판매합니다. 총 판매 금액을 계산해 사용자에게 확인을 받은 뒤 처리합니다.

```javascript
function claimCoinAid() {
  if (state.coins >= 100) return;
  state.coins += 5;
  saveState();
  renderAll();
}
```
코인이 100 미만일 때만 5코인을 지원받을 수 있는 구제 기능입니다.

```javascript
function resetGame() {
  if (!confirm('진행 상황을 모두 초기화할까요? 이 작업은 되돌릴 수 없습니다.')) return;
  state = defaultState();
  saveState();
  renderAll();
}
```
확인창을 거쳐 게임 진행 상황을 완전히 초기화합니다.

```javascript
const el = (sel) => document.querySelector(sel);

function rarityInfo(key) {
  return RARITIES.find((r) => r.key === key);
}
```
`document.querySelector`를 짧게 쓰기 위한 헬퍼와, 등급 키로 등급 정보를 찾는 헬퍼입니다.

```javascript
function renderHeader() {
  el('#coin-count').textContent = state.coins.toLocaleString();
  el('#box-opened-count').textContent = state.boxesOpened.toLocaleString();
  const discovered = Object.keys(state.dex).length;
  el('#dex-progress').textContent = `${discovered} / ${ALL_ITEMS.length}`;
  el('#coin-aid-btn').disabled = state.coins >= 100;
}
```
상단 통계(코인, 개봉수, 도감 진행도)를 현재 상태 값으로 갱신하고, 코인이 100 이상이면 구제 버튼을 비활성화합니다.

```javascript
function renderShop() {
  const wrap = el('#shop-boxes');
  wrap.innerHTML = '';
  for (const box of BOXES) {
    const unlocked = isUnlocked(box);
    const affordable = state.coins >= box.cost;
    const card = document.createElement('div');
    card.className = 'box-card' + (unlocked ? '' : ' locked');
    card.dataset.box = box.key;

    let unlockText = '';
    if (!unlocked && box.unlock.type === 'boxesOpened') {
      unlockText = `<div class="unlock-hint">${ICONS.lock} 상자 ${box.unlock.value}개 개봉 시 해금 (현재 ${state.boxesOpened})</div>`;
    }

    card.innerHTML = `
      <div class="box-icon-ring">${ICONS[box.icon]}</div>
      <div class="box-name">${box.label}</div>
      <div class="box-cost">${box.cost.toLocaleString()} 코인</div>
      ${unlockText}
      <button class="buy-btn" ${!unlocked || !affordable ? 'disabled' : ''}>
        ${unlocked ? '구매하기' : '잠김'}
      </button>
    `;
    if (unlocked) {
      card.querySelector('.buy-btn').addEventListener('click', () => buyBox(box.key));
    }
    wrap.appendChild(card);
  }
}
```
상점 탭에 상자 카드를 그립니다. 잠긴 상자는 해금 조건을 안내 문구로 보여주고, 구매 버튼은 잠겨있거나 코인이 부족하면 비활성화됩니다.

```javascript
function renderInventory() {
  const wrap = el('#inventory-list');
  const entries = Object.entries(state.inventory);
  el('#sell-all-inv-btn').disabled = entries.length === 0;
  if (entries.length === 0) {
    wrap.innerHTML = '<div class="empty-msg">보유한 아이템이 없습니다. 상자를 열어보세요!</div>';
    return;
  }
  wrap.innerHTML = '';
  entries
    .sort((a, b) => ITEM_BY_ID[b[0]].price - ITEM_BY_ID[a[0]].price)
    .forEach(([itemId, qty]) => {
      const item = ITEM_BY_ID[itemId];
      const rarity = rarityInfo(item.rarityKey);
      const row = document.createElement('div');
      row.className = 'inv-row';
      row.innerHTML = `
        <div class="inv-stamp" style="--r-color:${rarity.color}">${rarity.label[0]}</div>
        <div class="inv-info">
          <div class="inv-name">${item.name} <span class="inv-qty">x${qty}</span></div>
          <div class="inv-meta"><span style="color:${rarity.color}">${rarity.label}</span> · 개당 ${item.price.toLocaleString()} 코인</div>
        </div>
        <div class="inv-actions">
          <button class="sell-one-btn">1개 판매</button>
          <button class="sell-all-btn">전체 판매</button>
        </div>
      `;
      row.querySelector('.sell-one-btn').addEventListener('click', () => sellItem(itemId, 1));
      row.querySelector('.sell-all-btn').addEventListener('click', () => sellItem(itemId, qty));
      wrap.appendChild(row);
    });
}
```
인벤토리 탭을 그립니다. 아이템이 없으면 안내 문구를 보여주고, 있으면 비싼 순으로 정렬해 한 줄씩 표시하며 개별/전체 판매 버튼을 연결합니다.

```javascript
function renderDex() {
  const wrap = el('#dex-grid');
  wrap.innerHTML = '';
  const boxOrder = BOXES.map((b) => b.key);
  const sorted = [...ALL_ITEMS].sort((a, b) => {
    if (a.boxKey !== b.boxKey) return boxOrder.indexOf(a.boxKey) - boxOrder.indexOf(b.boxKey);
    const rOrder = RARITIES.map((r) => r.key);
    return rOrder.indexOf(a.rarityKey) - rOrder.indexOf(b.rarityKey);
  });
  for (const item of sorted) {
    const discovered = !!state.dex[item.id];
    const rarity = rarityInfo(item.rarityKey);
    const box = BOXES.find((b) => b.key === item.boxKey);
    const cell = document.createElement('div');
    cell.className = 'dex-cell' + (discovered ? '' : ' undiscovered');
    if (discovered) {
      cell.style.borderColor = rarity.color;
      cell.innerHTML = `
        <div class="dex-box-tag" style="color:${rarity.color}">${ICONS[box.icon]}</div>
        <div class="dex-name">${item.name}</div>
        <div class="dex-rarity" style="color:${rarity.color}">${rarity.label}</div>
        <div class="dex-price">${item.price.toLocaleString()}C</div>
      `;
    } else {
      cell.innerHTML = `
        <div class="dex-box-tag">${ICONS[box.icon]}</div>
        <div class="dex-name">???</div>
        <div class="dex-rarity">-</div>
        <div class="dex-price">-</div>
      `;
    }
    wrap.appendChild(cell);
  }
}
```
도감 탭을 그립니다. 전체 아이템을 상자→등급 순으로 정렬해서, 발견한 아이템은 정보를 보여주고 발견하지 못한 아이템은 물음표로 가려서 보여줍니다.

```javascript
function renderAll() {
  renderHeader();
  renderShop();
  renderInventory();
  renderDex();
}
```
상태가 바뀔 때마다 화면 전체(헤더, 상점, 인벤토리, 도감)를 다시 그리는 함수입니다.

```javascript
function showReveal(box, rarity, item, isNew, justUnlocked) {
  const modal = el('#reveal-modal');
  const card = el('#reveal-card');
  card.style.setProperty('--rarity-color', rarity.color);
  const isLegendary = rarity.key === 'legendary';

  card.querySelector('#reveal-serial').textContent = String(state.boxesOpened).padStart(6, '0');
  const stampEl = card.querySelector('#reveal-stamp');
  stampEl.textContent = rarity.label;
  stampEl.classList.toggle('is-legendary', isLegendary);
  card.querySelector('#reveal-box-label').innerHTML = `${ICONS[box.icon]} ${box.label}`;

  const rarityEl = card.querySelector('.reveal-rarity');
  rarityEl.textContent = rarity.label;
  rarityEl.classList.toggle('is-legendary', isLegendary);
  rarityEl.style.color = isLegendary ? '' : rarity.color;

  card.querySelector('.reveal-name').textContent = item.name;
  card.querySelector('.reveal-price').textContent = `판매가 ${item.price.toLocaleString()} 코인`;
  card.querySelector('.reveal-new').style.display = isNew ? 'block' : 'none';

  const unlockMsg = el('#reveal-unlock-msg');
  if (justUnlocked.length > 0) {
    unlockMsg.style.display = 'block';
    unlockMsg.textContent = `${justUnlocked.map((b) => b.label).join(', ')}이(가) 해금되었습니다`;
  } else {
    unlockMsg.style.display = 'none';
  }

  modal.classList.remove('hidden');
  card.classList.remove('pop');
  requestAnimationFrame(() => card.classList.add('pop'));
}

function hideReveal() {
  el('#reveal-modal').classList.add('hidden');
}
```
상자를 열었을 때 결과 티켓 모달을 채우고 보여주는 함수입니다. 일련번호, 등급 스탬프, 아이템 이름/가격, NEW 배지, 새로 해금된 상자 안내까지 채워 넣고, 애니메이션(`pop` 클래스)을 트리거해 등장 효과를 줍니다. `hideReveal`은 모달을 다시 숨깁니다.

```javascript
function initTabs() {
  document.querySelectorAll('.tab-btn').forEach((btn) => {
    btn.addEventListener('click', () => {
      document.querySelectorAll('.tab-btn').forEach((b) => b.classList.remove('active'));
      document.querySelectorAll('.tab-panel').forEach((p) => p.classList.remove('active'));
      btn.classList.add('active');
      el(`#tab-${btn.dataset.tab}`).classList.add('active');
    });
  });
}
```
탭 버튼 클릭 시, 모든 탭/패널의 `active` 클래스를 지운 뒤 클릭한 탭과 그에 맞는 패널에만 다시 `active`를 붙여 화면을 전환합니다.

```javascript
function init() {
  initTabs();
  el('#reveal-close').addEventListener('click', hideReveal);
  el('#reveal-modal').addEventListener('click', (e) => {
    if (e.target.id === 'reveal-modal') hideReveal();
  });
  el('#reset-btn').addEventListener('click', resetGame);
  el('#coin-aid-btn').addEventListener('click', claimCoinAid);
  el('#sell-all-inv-btn').addEventListener('click', sellAllInventory);
  renderAll();
}

document.addEventListener('DOMContentLoaded', init);
```
페이지가 로드되면 실행되는 초기화 함수입니다. 탭 전환, 모달 닫기(닫기 버튼 클릭 또는 바깥 영역 클릭), 초기화/구제/전체판매 버튼을 각각 연결하고, 마지막으로 화면 전체를 처음 한 번 그립니다.

---
