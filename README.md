<!DOCTYPE html>

<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<meta name="theme-color" content="#9333ea">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="Love Wrapped">
<meta name="description" content="Retrospectiva romântica do nosso amor">
<title>Love Wrapped 💕</title>
<script src="https://cdn.tailwindcss.com"></script>
<style>
@keyframes fade-in {
from { opacity: 0; transform: translateY(20px); }
to { opacity: 1; transform: translateY(0); }
}
@keyframes slide-up {
from { opacity: 0; transform: translateY(50px); }
to { opacity: 1; transform: translateY(0); }
}
@keyframes pulse-heart {
0%, 100% { transform: scale(1); }
50% { transform: scale(1.1); }
}
.animate-fade-in { animation: fade-in 0.8s ease-out forwards; }
.animate-slide-up { animation: slide-up 0.8s ease-out forwards; }
.animate-pulse-heart { animation: pulse-heart 1.5s ease-in-out infinite; }
* { -webkit-tap-highlight-color: transparent; }
body { overscroll-behavior: none; touch-action: pan-y; }
</style>
</head>
<body class="bg-gradient-to-br from-gray-900 via-purple-900 to-pink-900 text-white min-h-screen overflow-x-hidden">

<div id="app"></div>

<script>
const app = document.getElementById('app');
let currentSection = 0;
let showConfig = true;
let isViewMode = false;
let shareableLink = '';
let formData = { couple: '', startDate: '', message: '', photos: [], musicUrl: '' };
let timeData = {};
let timeInterval = null;

// Verificar URL para dados compartilhados
const urlParams = new URLSearchParams(window.location.search);
const idParam = urlParams.get('id');

if (idParam && window.storage) {
window.storage.get(idParam, true).then(result => {
if (result && result.value) {
try {
formData = JSON.parse(result.value);
showConfig = false;
isViewMode = true;
currentSection = 0;
render();
startTimeCounter();
} catch (error) {
console.error('Erro ao carregar:', error);
render();
}
} else {
render();
}
}).catch(() => render());
} else {
render();
}

function startTimeCounter() {
if (timeInterval) clearInterval(timeInterval);
if (formData.startDate) {
timeInterval = setInterval(() => {
const start = new Date(formData.startDate);
const now = new Date();
const diff = now - start;
timeData = {
years: Math.floor(diff / (1000 * 60 * 60 * 24 * 365)),
months: Math.floor((diff % (1000 * 60 * 60 * 24 * 365)) / (1000 * 60 * 60 * 24 * 30)),
days: Math.floor((diff % (1000 * 60 * 60 * 24 * 30)) / (1000 * 60 * 60 * 24)),
hours: Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60)),
minutes: Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60)),
seconds: Math.floor((diff % (1000 * 60)) / 1000),
totalDays: Math.floor(diff / (1000 * 60 * 60 * 24))
};
if (!showConfig) updateTimeDisplay();
}, 1000);
}
}

function updateTimeDisplay() {
document.querySelectorAll('[data-time]').forEach(el => {
const key = el.getAttribute('data-time');
if (timeData[key] !== undefined) {
el.textContent = String(timeData[key]).padStart(2, '0');
}
});
}

function handleFileUpload(e) {
Array.from(e.target.files).forEach(file => {
const reader = new FileReader();
reader.onload = (e) => {
formData.photos.push(e.target.result);
updatePhotoCount();
};
reader.readAsDataURL(file);
});
}

function updatePhotoCount() {
const photoCount = document.getElementById('photoCount');
if (photoCount) photoCount.textContent = `${formData.photos.length} foto(s) adicionada(s)`;
}

function startExperience() {
if (formData.couple && formData.startDate) {
showConfig = false;
currentSection = 0;
render();
startTimeCounter();
}
}

function nextSection() {
currentSection++;
render();
}

async function generateShareableLink() {
if (!window.storage) {
alert('Sistema de storage não disponível. Recarregue a página.');
return;
}
try {
const storageKey = `love-wrapped-${Date.now()}`;
await window.storage.set(storageKey, JSON.stringify(formData), true);
const baseUrl = window.location.origin + window.location.pathname;
shareableLink = `${baseUrl}?id=${storageKey}`;
showShareModal();
} catch (error) {
alert('Erro ao gerar link. Tente novamente.');
}
}

function showShareModal() {
const modal = document.getElementById('shareModal');
if (modal) modal.classList.remove('hidden');
}

function closeShareModal() {
const modal = document.getElementById('shareModal');
if (modal) modal.classList.add('hidden');
}

function copyToClipboard() {
if (navigator.clipboard) {
navigator.clipboard.writeText(shareableLink).then(() => {
alert('✅ Link copiado! 💕');
}).catch(fallbackCopy);
} else {
fallbackCopy();
}
}

function fallbackCopy() {
const textArea = document.createElement('textarea');
textArea.value = shareableLink;
textArea.style.position = 'fixed';
textArea.style.left = '-999999px';
document.body.appendChild(textArea);
textArea.select();
try {
document.execCommand('copy');
alert('✅ Link copiado! 💕');
} catch (err) {
alert('Copie manualmente o link');
}
document.body.removeChild(textArea);
}

function shareViaWhatsApp() {
const message = `💕 Olá amor! Fiz uma surpresa especial pra você! ${shareableLink}`;
window.open(`https://wa.me/?text=${encodeURIComponent(message)}`, '_blank');
}

function createNewWrapped() {
formData = { couple: '', startDate: '', message: '', photos: [], musicUrl: '' };
isViewMode = false;
showConfig = true;
currentSection = 0;
if (timeInterval) clearInterval(timeInterval);
window.history.pushState({}, '', window.location.pathname);
render();
}

function render() {
if (showConfig) {
app.innerHTML = renderConfig();
attachConfigListeners();
} else {
app.innerHTML = renderSection(currentSection);
}
}

function renderConfig() {
return `
<div class="max-w-2xl mx-auto p-6 pt-12">
<div class="text-center mb-12 animate-fade-in">
<svg class="w-16 h-16 text-pink-400 mx-auto mb-4 animate-pulse-heart" fill="currentColor" viewBox="0 0 24 24">
<path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/>
</svg>
<h1 class="text-4xl md:text-5xl font-bold mb-4 bg-gradient-to-r from-pink-400 to-purple-400 bg-clip-text text-transparent">Love Wrapped</h1>
<p class="text-xl text-white/80">Configure sua retrospectiva do amor 💕</p>
</div>
<div class="bg-white/10 backdrop-blur-xl rounded-3xl p-8 border border-white/20 space-y-6 animate-slide-up">
<div>
<label class="block text-sm font-semibold mb-2">Nome do Casal</label>
<input type="text" id="couple" placeholder="Ex: Maria & João" value="${formData.couple}" class="w-full bg-white/10 border border-white/30 rounded-xl px-4 py-3 text-white placeholder-white/50 focus:outline-none focus:border-pink-400 transition-colors"/>
</div>
<div>
<label class="block text-sm font-semibold mb-2">Data de Início do Relacionamento</label>
<input type="date" id="startDate" value="${formData.startDate}" class="w-full bg-white/10 border border-white/30 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-pink-400 transition-colors"/>
</div>
<div>
<label class="block text-sm font-semibold mb-2">Mensagem Especial</label>
<textarea id="message" placeholder="Escreva uma mensagem romântica..." rows="4" class="w-full bg-white/10 border border-white/30 rounded-xl px-4 py-3 text-white placeholder-white/50 focus:outline-none focus:border-pink-400 transition-colors resize-none">${formData.message}</textarea>

```
</div>
<div>
<label class="block text-sm font-semibold mb-2">Fotos do Casal</label>
<label class="flex flex-col items-center justify-center w-full h-32 border-2 border-dashed border-white/30 rounded-xl cursor-pointer hover:border-pink-400 transition-colors bg-white/5">
<svg class="w-8 h-8 text-pink-400 mb-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12"/>
</svg>
<span class="text-sm text-white/70">Clique para adicionar fotos</span>
<input type="file" id="photoUpload" multiple accept="image/*" class="hidden"/>
</label>
<p id="photoCount" class="text-sm text-pink-400 mt-2">${formData.photos.length} foto(s) adicionada(s)</p>
</div>
<div>
<label class="block text-sm font-semibold mb-2">🎵 Link da Música (Spotify/YouTube)</label>
<input type="url" id="musicUrl" placeholder="Cole o link da música aqui" value="${formData.musicUrl}" class="w-full bg-white/10 border border-white/30 rounded-xl px-4 py-3 text-white placeholder-white/50 focus:outline-none focus:border-pink-400 transition-colors"/>
</div>
<button onclick="startExperience()" id="startBtn" class="w-full bg-gradient-to-r from-pink-500 to-purple-500 text-white py-4 rounded-xl font-semibold text-lg hover:scale-105 transition-transform disabled:opacity-50 disabled:cursor-not-allowed">Criar Minha Retrospectiva 💕</button>
</div>
</div>
`;
}

function attachConfigListeners() {
document.getElementById('couple').addEventListener('input', (e) => {
formData.couple = e.target.value;
updateStartButton();
});
document.getElementById('startDate').addEventListener('change', (e) => {
formData.startDate = e.target.value;
updateStartButton();
});
document.getElementById('message').addEventListener('input', (e) => {
formData.message = e.target.value;
});
document.getElementById('musicUrl').addEventListener('input', (e) => {
formData.musicUrl = e.target.value;
});
document.getElementById('photoUpload').addEventListener('change', handleFileUpload);
updateStartButton();
}

function updateStartButton() {
const btn = document.getElementById('startBtn');
if (btn) btn.disabled = !formData.couple || !formData.startDate;
}

function renderSection(section) {
const sections = [
renderIntroSection(),
renderTimeSection(),
renderStatsSection(),
renderMessageSection(),
renderGallerySection(),
renderFinalSection()
];
return `
<div class="min-h-screen flex items-center justify-center p-6">
${sections[section]}
</div>
${renderShareModal()}
${isViewMode ? '<button onclick="createNewWrapped()" class="fixed bottom-6 left-6 bg-white/20 backdrop-blur text-white px-6 py-3 rounded-full font-semibold hover:bg-white/30 transition-all z-40">← Criar Minha Própria</button>' : ''}
`;
}

function renderIntroSection() {
return `
<div class="text-center animate-fade-in">
<svg class="w-20 h-20 text-pink-400 mx-auto mb-8 animate-pulse-heart" fill="currentColor" viewBox="0 0 24 24">
<path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/>
</svg>
<h1 class="text-5xl md:text-7xl font-bold mb-6 bg-gradient-to-r from-pink-400 via-purple-400 to-blue-400 bg-clip-text text-transparent">Sim amor,</h1>
<h2 class="text-3xl md:text-5xl font-bold mb-8 text-white">eu fiz uma retrospectiva<br/>do nosso relacionamento ❤️🥹</h2>
<button onclick="nextSection()" class="bg-gradient-to-r from-pink-500 to-purple-500 text-white px-12 py-4 rounded-full text-xl font-semibold hover:scale-105 transition-transform shadow-lg inline-flex items-center gap-2">
Vamos lá 💕
<svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 7l5 5m0 0l-5 5m5-5H6"/>
</svg>
</button>
</div>
`;
}

function renderTimeSection() {
return `
<div class="text-center animate-slide-up max-w-4xl w-full">
<h2 class="text-4xl md:text-6xl font-bold mb-12 text-white">Nosso Tempo Juntos</h2>
<div class="bg-white/10 backdrop-blur-lg rounded-3xl p-12 mb-8 border border-white/20">
<div class="text-6xl md:text-8xl font-bold bg-gradient-to-r from-pink-400 to-purple-400 bg-clip-text text-transparent mb-4">
<span data-time="years">${timeData.years || 0}</span> anos, <span data-time="months">${timeData.months || 0}</span> meses
</div>
<div class="text-4xl md:text-5xl font-bold text-white/90 mb-8">e <span data-time="days">${timeData.days || 0}</span> dias</div>
<div class="grid grid-cols-3 gap-6 text-2xl md:text-3xl font-mono text-pink-300">
<div><div class="font-bold" data-time="hours">${String(timeData.hours || 0).padStart(2, '0')}</div><div class="text-sm text-white/60">horas</div></div>
<div><div class="font-bold" data-time="minutes">${String(timeData.minutes || 0).padStart(2, '0')}</div><div class="text-sm text-white/60">minutos</div></div>
<div><div class="font-bold" data-time="seconds">${String(timeData.seconds || 0).padStart(2, '0')}</div><div class="text-sm text-white/60">segundos</div></div>
</div>
</div>
<button onclick="nextSection()" class="bg-white/20 backdrop-blur text-white px-8 py-3 rounded-full font-semibold hover:bg-white/30 transition-all">Continuar →</button>
</div>
`;
}

function renderStatsSection() {
return `
<div class="max-w-4xl w-full animate-fade-in text-center">
<h2 class="text-4xl md:text-6xl font-bold mb-16 text-white">Em ${timeData.totalDays || 0} dias juntos...</h2>
<div class="space-y-8">
<div class="transform hover:scale-105 transition-transform">
<div class="text-6xl md:text-8xl font-bold bg-gradient-to-r from-pink-400 to-rose-400 bg-clip-text text-transparent">1.240.129</div>
<div class="text-2xl md:text-3xl text-white/80 mt-2">mensagens trocadas 💬</div>
</div>
<div class="transform hover:scale-105 transition-transform">
<div class="text-6xl md:text-8xl font-bold bg-gradient-to-r from-purple-400 to-pink-400 bg-clip-text text-transparent">2.503.847</div>
<div class="text-2xl md:text-3xl text-white/80 mt-2">risadas compartilhadas 😂</div>
</div>
<div class="transform hover:scale-105 transition-transform">
<div class="text-6xl md:text-8xl font-bold bg-gradient-to-r from-blue-400 to-purple-400 bg-clip-text text-transparent">1.876.921</div>
<div class="text-2xl md:text-3xl text-white/80 mt-2">abraços apertados 🤗</div>
</div>
</div>
<button onclick="nextSection()" class="mt-12 bg-white/20 backdrop-blur text-white px-8 py-3 rounded-full font-semibold hover:bg-white/30 transition-all">Continuar →</button>
</div>
`;
}

function renderMessageSection() {
return `
<div class="max-w-3xl animate-fade-in text-center">
<h2 class="text-4xl md:text-6xl font-bold mb-12 text-white">Uma Mensagem Especial 💌</h2>
<div class="bg-gradient-to-br from-pink-500/20 to-purple-500/20 backdrop-blur-xl rounded-3xl p-12 border-2 border-white/30 shadow-2xl">
<p class="text-2xl md:text-3xl text-white leading-relaxed font-light">${formData.message || "Você é o amor da minha vida e a pessoa que me faz querer ser melhor a cada novo dia 💖"}</p>
</div>
<button onclick="nextSection()" class="mt-12 bg-gradient-to-r from-pink-500 to-purple-500 text-white px-10 py-4 rounded-full text-lg font-semibold hover:scale-105 transition-transform">Ver Galeria de Fotos →</button>
</div>
`;
}

function renderGallerySection() {
let photosHTML = formData.photos.length > 0
? `<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 mb-8">${formData.photos.map((photo, idx) => `
<div class="relative overflow-hidden rounded-2xl aspect-square group animate-fade-in" style="animation-delay: ${idx * 150}ms">
<img src="${photo}" alt="Momento ${idx + 1}" class="w-full h-full object-cover transition-transform duration-500 group-hover:scale-110"/>
<div class="absolute inset-0 bg-gradient-to-t from-black/60 to-transparent opacity-0 group-hover:opacity-100 transition-opacity duration-300 flex items-end justify-center pb-4">
<span class="text-white font-semibold">Momento ${idx + 1} 💕</span>
</div>
</div>
`).join('')}</div>`
: `<div class="bg-white/10 backdrop-blur rounded-3xl p-12 border border-white/20 mb-8">
<svg class="w-20 h-20 text-pink-400 mx-auto mb-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 9a2 2 0 012-2h.93a2 2 0 001.664-.89l.812-1.22A2 2 0 0110.07 4h3.86a2 2 0 011.664.89l.812 1.22A2 2 0 0018.07 7H19a2 2 0 012 2v9a2 2 0 01-2 2H5a2 2 0 01-2-2V9z"/>
<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 13a3 3 0 11-6 0 3 3 0 016 0z"/>
</svg>
<p class="text-xl text-white/80">Nenhuma foto foi adicionada ainda 📷</p>
</div>`;
return `
<div class="max-w-5xl w-full animate-fade-in text-center">
<h2 class="text-4xl md:text-6xl font-bold mb-12 text-white">Nossos Melhores Momentos 📸</h2>
${photosHTML}
<button onclick="nextSection()" class="bg-white/20 backdrop-blur text-white px-8 py-3 rounded-full font-semibold hover:bg-white/30 transition-all">Continuar →</button>
</div>
`;
}

function renderFinalSection() {
return `
<div class="text-center animate-fade-in">
<div class="mb-8">
<svg class="w-32 h-32 text-pink-500 mx-auto animate-pulse-heart" fill="currentColor" viewBox="0 0 24 24">
<path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/>
</svg>
</div>
<h1 class="text-5xl md:text-8xl font-bold mb-8 bg-gradient-to-r from-pink-400 via-red-400 to-purple-400 bg-clip-text text-transparent">Eu te amo</h1>
<h2 class="text-4xl md:text-6xl font-bold text-white mb-12">mais que tudo! 💕</h2>
<p class="text-xl md:text-2xl text-white/80 mb-12 max-w-2xl mx-auto">Obrigado por cada momento, cada riso, cada abraço.<br/>Você faz minha vida completa. ❤️</p>
<button onclick="generateShareableLink()" class="bg-gradient-to-r from-pink-500 to-purple-500 text-white px-10 py-4 rounded-full text-xl font-semibold hover:scale-105 transition-transform shadow-lg inline-flex items-center gap-2">
<svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8.684 13.342C8.886 12.938 9 12.482 9 12c0-.482-.114-.938-.316-1.342m0 2.684a3 3 0 110-2.684m0 2.684l6.632 3.316m-6.632-6l6.632-3.316m0 0a3 3 0 105.367-2.684 3 3 0 00-5.367 2.684zm0 9.316a3 3 0 105.368 2.684 3 3 0 00-5.368-2.684z"/>
</svg>
Compartilhar com meu amor 💌
</button>
</div>
`;
}

function renderShareModal() {
return `
<div id="shareModal" class="hidden fixed inset-0 bg-black/80 backdrop-blur-sm flex items-center justify-center p-6 z-50 animate-fade-in">
<div class="bg-gradient-to-br from-pink-500/20 to-purple-500/20 backdrop-blur-xl rounded-3xl p-8 max-w-lg w-full border-2 border-white/30 shadow-2xl">
<div class="text-center mb-6">
<svg class="w-16 h-16 text-pink-400 mx-auto mb-4 animate-pulse-heart" fill="currentColor" viewBox="0 0 24 24">
<path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/>
</svg>
<h3 class="text-3xl font-bold mb-2">Link Gerado! 💕</h3>
<p class="text-white/80">Compartilhe essa retrospectiva com seu amor</p>
</div>
<div class="bg-white/10 rounded-xl p-4 mb-6 border border-white/20">
<p class="text-sm text-white/60 mb-2">Seu link:</p>
<p id="shareLinkText" class="text-white font-mono text-xs break-all">${shareableLink}</p>
</div>
<div class="space-y-3">
<button onclick="copyToClipboard()" class="w-full bg-white/20 hover:bg-white/30 text-white py-3
```
