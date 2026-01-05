<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Gallery + Spoiler + Lightbox</title>

    <style>
        :root {
            --gallery-gap: 10px;
            --item-h: 180px;
            --radius: 12px;
            --controls-h: 180px; /* altura para setas+thumbs+legenda */
            --ring: #3b82f6;     /* azul do foco */
            --safe-top: env(safe-area-inset-top, 0px);
            --safe-right: env(safe-area-inset-right, 0px);
            --safe-bottom: env(safe-area-inset-bottom, 0px);
            --safe-left: env(safe-area-inset-left, 0px);
        }

        /* ===== GALLERY ===== */
        .gallery {
            display: flex;
            flex-wrap: wrap;
            gap: var(--gallery-gap);
            justify-content: center;
            padding: 16px;
        }

        .gallery-item {
            position: relative;
            height: var(--item-h);
            width: auto;
            cursor: pointer;
            overflow: hidden;
            border-radius: var(--radius);

            display: flex;
            align-items: center;
            justify-content: center;
        }

        .gallery-item img {
            height: 100%;
            width: auto;
            display: block;
            transition: transform .35s ease;
            border-radius: var(--radius);
        }

        .gallery-item:hover img { transform: scale(1.08); }

        .title {
            position: absolute;
            bottom: 0; left: 0; right: 0;
            padding: 6px 10px;
            font-size: 14px;
            color: #fff;
            background: linear-gradient(to top, rgba(0,0,0,.8), transparent);
            opacity: 0;
            transition: opacity .25s;
            pointer-events: none;
        }
        .gallery-item:hover .title { opacity: 1; }

        /* +alt (canto inferior esquerdo) */
        .alt-badge {
            position: absolute;
            bottom: 6px;
            left: 6px;
            font-size: 12px;
            padding: 2px 6px;
            background: rgba(0,0,0,.65);
            color: #fff;
            border-radius: 4px;
            pointer-events: none;
        }

        /* ===== SPOILER ===== */
        @keyframes spoilerFade { from {opacity: 0;} to {opacity: 1;} }

        .spoiler {
            animation: spoilerFade .25s ease;
            position: absolute;
            inset: 0;
            z-index: 3;
            display: flex;
            align-items: center;
            justify-content: center;
            text-align: center;
            color: #fff;

            -webkit-backdrop-filter: blur(18px);
            backdrop-filter: blur(18px);
            background: radial-gradient(circle at center, rgba(0,0,0,.35), rgba(0,0,0,.75));
            border-radius: inherit;
            transition: opacity .25s ease, visibility .25s ease;
            padding: 16px;
        }
        .spoiler.hidden { opacity: 0; pointer-events: none; visibility: hidden; }

        .spoiler-inner {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 12px;
            max-width: min(90%, 640px);
        }

        .spoiler-icon { opacity: .95; }
        .eye-off-icon { width: 42px; height: 42px; color: #a1a1aa; }

        .spoiler-title { font-size: 18px; font-weight: 700; }
        .spoiler-sub { font-size: 14px; opacity: .9; }

        .spoiler-view {
            padding: 10px 18px;
            border-radius: 999px;
            border: none;
            background: #fff;
            color: #111;
            font-size: 14px;
            font-weight: 700;
            cursor: pointer;
            display: inline-flex;
            align-items: center;
            gap: 8px;
        }
        .spoiler-view:hover { background: #f4f4f5; }
        .eye-on-icon { width: 18px; height: 18px; }

        /* Hide (top-right) */
        .spoiler-hide {
            position: absolute;
            top: 8px;
            right: 8px;
            z-index: 6;
            display: none; /* aparece quando revelado */
            align-items: center;
            gap: 6px;
            padding: 6px 10px;
            font-size: 12px;
            font-weight: 700;
            background: rgba(0,0,0,.65);
            color: #fff;
            border: none;
            border-radius: 999px;
            cursor: pointer;
            -webkit-backdrop-filter: blur(6px);
            backdrop-filter: blur(6px);
            transition: opacity .2s ease, transform .2s ease;
        }
        .spoiler-hide svg { width: 16px; height: 16px; }

        /* ===== LIGHTBOX ===== */
        .lightbox {
            position: fixed;
            inset: 0;
            background: rgba(0,0,0,.9);
            display: none;
            align-items: center;
            justify-content: center;
            z-index: 9999;
        }
        .lightbox.active { display: flex; }
        /* Evita flicker: oculta a imagem enquanto preparamos overlay/estado */
        .lightbox.preload .lb-wrap { visibility: hidden; }

        .lightbox-content {
            position: relative;
            width: min(96vw, 1400px);
            height: 96vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            border-radius: 12px;
            overflow: hidden;

            /* Margens de área segura (notch / UI do navegador mobile) */
            padding-top: calc(var(--safe-top) + 8px);
            padding-right: calc(var(--safe-right) + 8px);
            padding-bottom: calc(var(--safe-bottom) + 8px);
            padding-left: calc(var(--safe-left) + 8px);
        }

        .lb-stage {
            position: relative;
            flex: 1 1 auto;
            width: 100%;
            min-height: 0;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        /* Wrapper com o TAMANHO EXATO da imagem renderizada */
        .lb-wrap {
            position: relative;
            display: inline-block;
            line-height: 0;
            border-radius: 8px;
            overflow: hidden; /* recorta spoiler no raio da imagem */
        }
        .lb-wrap img {
            display: block;
            width: 100%;
            height: 100%;
            object-fit: contain;
            border-radius: inherit;
        }

        /* Barra de controles abaixo da imagem (setas + thumbs + legenda) */
        .lb-controlsbar {
            width: 100%;
            min-height: var(--controls-h);
            padding: 10px 12px 12px 12px;
            display: grid;
            grid-template-areas:
                "prev thumbs next"
                "caption caption caption";
            grid-template-columns: auto 1fr auto;
            align-items: center;
            gap: 12px;
        }

        .lb-btn, .lb-close {
            font-size: 22px;
            padding: 10px 14px;
            border: none;
            cursor: pointer;
            background: rgba(255,255,255,.12);
            color: #fff;
            border-radius: 999px;
        }
        .lb-btn:hover, .lb-close:hover { background: rgba(255,255,255,.22); }
        .lb-prev { grid-area: prev; }
        .lb-next { grid-area: next; }

        .alt-strip {
            grid-area: thumbs;
            display: flex;
            gap: 8px;
            align-items: center;
            justify-content: center;
            min-height: 72px;
            flex-wrap: wrap;
        }
        .alt-strip img {
            height: 64px;
            width: auto;
            border-radius: 8px;
            cursor: pointer;
            opacity: .7;
            transition: opacity .2s ease, transform .2s ease, box-shadow .2s ease;
        }
        .alt-strip img:hover { opacity: 1; transform: translateY(-2px); }
        .alt-strip img.active {
            opacity: 1;
            transform: translateY(-2px);
            box-shadow: 0 0 0 2px #fff, 0 0 0 4px var(--ring);
        }

        .lb-caption {
            grid-area: caption;
            margin-top: 6px;
            font-size: 14px;
            color: #ddd;
            text-align: center;
        }

        .lb-close {
            position: absolute;
            top: calc(8px + var(--safe-top));
            right: calc(8px + var(--safe-right));
            font-size: 22px;
            z-index: 12;
        }

        /* Foco visível (acessibilidade) */
        .lb-btn:focus-visible,
        .lb-close:focus-visible,
        .alt-strip img:focus-visible,
        .spoiler-view:focus-visible,
        .spoiler-hide:focus-visible,
        .gallery-item:focus-visible {
            outline: 2px solid var(--ring);
            outline-offset: 2px;
        }

        @media (max-width: 600px) {
            :root { --item-h: 180px; --controls-h: 200px; }

            /* Thumbs com scroll horizontal para não empurrar a imagem */
            .alt-strip {
                justify-content: flex-start;
                flex-wrap: nowrap;
                overflow-x: auto;
                overflow-y: hidden;
                -webkit-overflow-scrolling: touch;
                scrollbar-width: thin;
            }
            .alt-strip img { flex: 0 0 auto; height: 64px; }

            .spoiler-view { font-size: 15px; padding: 12px 20px; }
            .spoiler-title { font-size: 20px; }
            .spoiler-sub { font-size: 15px; }
            .eye-off-icon { width: 52px; height: 52px; }
            .spoiler-hide { padding: 10px 14px; font-size: 14px; }
            .spoiler-hide svg { width: 18px; height: 18px; }
        }
    </style>
</head>
<body>

    <!-- GALLERY (exemplos) -->
    <div class="gallery">
        <div class="gallery-item"
             data-full="https://placehold.co/1200x800?text=spoiler+version+A"
             data-alt="https://placehold.co/1200x800?text=spoiler+version+A,https://placehold.co/1200x800?text=spoiler+version+B,https://placehold.co/1200x800?text=spoiler+version+C"
             data-title="Spoiler Image"
             data-spoiler="true"
             data-spoiler-sub="Sensitive Content">
            <img loading="lazy" src="https://placehold.co/600x400?text=spoiler+version+A" alt="Spoiler Image">
            <div class="title">Spoiler Image</div>
        </div>

        <div class="gallery-item"
             data-full="https://placehold.co/700x1200?text=1+version+A"
             data-alt="https://placehold.co/700x1200?text=1+version+A,https://placehold.co/700x1200?text=1+version+B"
             data-title="Normal Image 1">
            <img loading="lazy" src="https://placehold.co/350x600?text=1+version+A" alt="Normal Image 1">
            <div class="title">Normal Image 1</div>
        </div>

        <div class="gallery-item"
             data-full="https://placehold.co/1200x700?text=2"
             data-title="Normal Image 2">
            <img loading="lazy" src="https://placehold.co/600x350?text=2" alt="Normal Image 2">
            <div class="title">Normal Image 2</div>
        </div>
    </div>

    <!-- LIGHTBOX -->
    <div class="lightbox" role="dialog" aria-modal="true" aria-hidden="true" aria-labelledby="lb-caption">
        <div class="lightbox-content">
            <button class="lb-close" aria-label="Close">✕</button>

            <div class="lb-stage">
                <div class="lb-wrap">
                    <img alt="">
                    <!-- Spoiler/Hide são injetados aqui -->
                </div>
            </div>

            <div class="lb-controlsbar">
                <button class="lb-btn lb-prev" aria-label="Previous image">‹ Prev</button>
                <div class="alt-strip"></div>
                <button class="lb-btn lb-next" aria-label="Next image">Next ›</button>
                <div class="lb-caption" id="lb-caption"></div>
            </div>
        </div>
    </div>

    <!-- TEMPLATE: Spoiler -->
    <template id="tpl-spoiler">
        <div class="spoiler">
            <div class="spoiler-inner">
                <div class="spoiler-icon">
                    <!-- Olho cortado (estilo X/Twitter) -->
                    <svg xmlns="http://www.w3.org/2000/svg" class="eye-off-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                        <path d="M10.733 5.076a10.744 10.744 0 0 1 11.205 6.575 1 1 0 0 1 0 .696 10.747 10.747 0 0 1-1.444 2.49"/>
                        <path d="M14.084 14.158a3 3 0 0 1-4.242-4.242"/>
                        <path d="M17.479 17.499a10.75 10.75 0 0 1-15.417-5.151 1 1 0 0 1 0-.696 10.75 10.75 0 0 1 4.446-5.143"/>
                        <path d="m2 2 20 20"/>
                    </svg>
                </div>
                <div class="spoiler-title">Content Warning</div>
                <div class="spoiler-sub">Sensitive Content</div>
                <button class="spoiler-view" type="button" aria-label="Show content">
                    <!-- Olho aberto -->
                    <svg xmlns="http://www.w3.org/2000/svg" class="eye-on-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                        <path d="M2.062 12.348a1 1 0 0 1 0-.696 10.75 10.75 0 0 1 19.876 0 1 1 0 0 1 0 .696 10.75 10.75 0 0 1-19.876 0"/>
                        <circle cx="12" cy="12" r="3"/>
                    </svg>
                    Show Content
                </button>
            </div>
        </div>
    </template>

    <!-- TEMPLATE: Hide -->
    <template id="tpl-hide">
        <button class="spoiler-hide" type="button" aria-label="Hide content">
            <svg xmlns="http://www.w3.org/2000/svg" class="eye-off-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M10.733 5.076a10.744 10.744 0 0 1 11.205 6.575 1 1 0 0 1 0 .696 10.747 10.747 0 0 1-1.444 2.49"/>
                <path d="M14.084 14.158a3 3 0 0 1-4.242-4.242"/>
                <path d="M17.479 17.499a10.75 10.75 0 0 1-15.417-5.151 1 1 0 0 1 0-.696 10.75 10.75 0 0 1 4.446-5.143"/>
                <path d="m2 2 20 20"/>
            </svg>
            Hide
        </button>
    </template>

    <script>
        /* ===== Utils ===== */
        function $(sel, ctx = document) { return ctx.querySelector(sel); }
        function $$(sel, ctx = document) { return Array.from(ctx.querySelectorAll(sel)); }

        const items = $$('.gallery-item');
        const lightbox = $('.lightbox');
        const lbStage = $('.lb-stage', lightbox);
        const lbWrap = $('.lb-wrap', lightbox);
        const lbImg = $('img', lbWrap);
        const lbStrip = $('.alt-strip', lightbox);
        const lbCaption = $('.lb-caption', lightbox);
        const lbPrev = $('.lb-prev', lightbox);
        const lbNext = $('.lb-next', lightbox);
        const lbClose = $('.lb-close', lightbox);

        /* Tornar itens focáveis para teclado */
        items.forEach(it => it.setAttribute('tabindex', '0'));

        let index = 0;
        let lbAllSrcs = [];
        let openerEl = null;
        let trapHandler = null;

        /* ===== Fontes ===== */
        function getSources(item) {
            const full = item.dataset.full || $('img', item)?.src || '';
            const alts = (item.dataset.alt || '').split(',').map(s => s.trim()).filter(Boolean);
            return { full, alts };
        }

        /* ===== +alt badge ===== */
        function ensureAltBadge(item) {
            if (item.dataset.alt && !$('.alt-badge', item)) {
                const b = document.createElement('div');
                b.className = 'alt-badge';
                b.textContent = '+alt';
                item.appendChild(b);
            }
        }

        /* ===== Spoiler ===== */
        function buildSpoilerOverlay(item) {
            const tpl = $('#tpl-spoiler');
            const node = tpl.content.firstElementChild.cloneNode(true);
            const sub = $('.spoiler-sub', node);
            sub.textContent = item.dataset.spoilerSub || 'Sensitive Content';
            return node;
        }
        function buildHideButton() {
            const tpl = $('#tpl-hide');
            return tpl.content.firstElementChild.cloneNode(true);
        }
        function syncSpoilerUI(item, container) {
            const revealed = item.dataset.revealed === 'true';
            const spoiler = $('.spoiler', container);
            const hideBtn = $('.spoiler-hide', container);
            if (!spoiler || !hideBtn) return;
            spoiler.classList.toggle('hidden', revealed);
            hideBtn.style.display = revealed ? 'flex' : 'none';
        }
        function attachSpoilerLogic(item, container) {
            let spoiler = $('.spoiler', container);
            let hideBtn = $('.spoiler-hide', container);
            if (!spoiler) container.appendChild(spoiler = buildSpoilerOverlay(item));
            if (!hideBtn) container.appendChild(hideBtn = buildHideButton());

            const viewBtn = $('.spoiler-view', container);
            if (viewBtn) {
                viewBtn.onclick = e => {
                    e.stopPropagation();
                    item.dataset.revealed = 'true';
                    syncSpoilerUI(item, container);
                    syncSpoilerUI(item, item);
                    syncSpoilerUI(item, lbWrap);
                };
            }
            hideBtn.onclick = e => {
                e.stopPropagation();
                delete item.dataset.revealed;
                syncSpoilerUI(item, container);
                syncSpoilerUI(item, item);
                syncSpoilerUI(item, lbWrap);
            };
            syncSpoilerUI(item, container);
        }

        items.forEach(item => {
            ensureAltBadge(item);
            if (item.dataset.spoiler === 'true') attachSpoilerLogic(item, item);
        });

        /* ===== Fit: imagem cabe no stage ===== */
        function fitImageToStage() {
            const stageW = lbStage.clientWidth;
            const stageH = lbStage.clientHeight;
            const natW = lbImg.naturalWidth || 1;
            const natH = lbImg.naturalHeight || 1;
            const scale = Math.min(stageW / natW, stageH / natH);
            const newW = Math.max(1, Math.floor(natW * scale));
            const newH = Math.max(1, Math.floor(natH * scale));
            lbWrap.style.width = newW + 'px';
            lbWrap.style.height = newH + 'px';
        }

        /* ===== Lightbox ===== */
        function renderLightboxSpoiler(item) {
            $('.spoiler', lbWrap)?.remove();
            $('.spoiler-hide', lbWrap)?.remove();
            if (item.dataset.spoiler === 'true') attachSpoilerLogic(item, lbWrap);
        }

        function openLightbox(i) {
            index = i;
            const item = items[index];
            const { full, alts } = getSources(item);

            // Entra já em 'preload' para evitar flicker de spoiler
            lightbox.classList.add('active', 'preload');
            lightbox.setAttribute('aria-hidden', 'false');
            document.body.style.overflow = 'hidden';

            // Monta spoiler no lightbox ANTES da imagem aparecer
            renderLightboxSpoiler(item);
            syncSpoilerUI(item, lbWrap); // garante estado inicial correto

            // Prepara thumbs
            lbStrip.innerHTML = '';
            lbAllSrcs = [full, ...alts.filter(s => s !== full)];
            lbAllSrcs.forEach((src, j) => {
                const t = document.createElement('img');
                t.src = src;
                t.alt = 'Alternative ' + (j + 1);
                t.setAttribute('tabindex', '0');
                t.addEventListener('click', e => { e.stopPropagation(); setActiveThumb(j); lbImg.src = src; });
                t.addEventListener('keydown', e => { if (e.key === 'Enter' || e.key === ' ') { e.preventDefault(); setActiveThumb(j); lbImg.src = src; }});
                if (j === 0) t.classList.add('active');
                lbStrip.appendChild(t);
            });

            // Seta imagem principal
            lbImg.onload = () => {
                fitImageToStage();
                // Agora que está tudo pronto, libera a visibilidade
                lightbox.classList.remove('preload');
                // Trap de foco
                openerEl = document.activeElement;
                const initial = lbClose || lightbox;
                enableFocusTrap(lightbox);
                initial.focus();
            };
            lbImg.src = full;
            lbImg.alt = item.dataset.title || '';
            lbCaption.textContent = item.dataset.title || '';

            if (lbImg.complete && lbImg.naturalWidth) {
                // Em alguns casos o cache dispara imediato
                lbImg.onload();
            }
        }

        function closeLightbox() {
            disableFocusTrap(lightbox);
            lightbox.classList.remove('active', 'preload');
            lightbox.setAttribute('aria-hidden', 'true');
            document.body.style.overflow = '';
            if (openerEl && typeof openerEl.focus === 'function') openerEl.focus();
            openerEl = null;
        }

        lbPrev.onclick = e => { e.stopPropagation(); index = (index - 1 + items.length) % items.length; openLightbox(index); };
        lbNext.onclick = e => { e.stopPropagation(); index = (index + 1) % items.length; openLightbox(index); };
        lbClose.onclick = () => closeLightbox();

        lightbox.addEventListener('click', e => { if (e.target === lightbox) closeLightbox(); });

        document.addEventListener('keydown', e => {
            if (!lightbox.classList.contains('active')) return;
            if (e.key === 'Escape') closeLightbox();
            if (e.key === 'ArrowRight') lbNext.click();
            if (e.key === 'ArrowLeft') lbPrev.click();
        });

        // Swipe
        let startX = 0;
        lightbox.addEventListener('touchstart', e => { startX = e.touches[0].clientX; });
        lightbox.addEventListener('touchend', e => {
            const dx = e.changedTouches[0].clientX - startX;
            if (dx > 50) lbPrev.click();
            if (dx &lt; -50) lbNext.click();
        });

        // Abrir por clique/teclado (Enter/Espaço)
        items.forEach((item, i) => {
            item.addEventListener('click', e => {
                if (e.target.closest('.spoiler-view') || e.target.closest('.spoiler-hide')) return;
                openLightbox(i);
            });
            item.addEventListener('keydown', e => {
                if (e.key === 'Enter' || e.key === ' ') {
                    e.preventDefault();
                    openLightbox(i);
                }
            });
        });

        /* ===== Miniatura ativa ===== */
        function setActiveThumb(i) {
            const thumbs = $$('.lightbox .alt-strip img');
            thumbs.forEach((t, idx) => t.classList.toggle('active', idx === i));
        }

        /* ===== Trap de foco ===== */
        const FOCUS_SEL = 'button,[href],input,select,textarea,[tabindex]:not([tabindex="-1"])';
        function getFocusable(root) {
            return Array.from(root.querySelectorAll(FOCUS_SEL))
                .filter(el => !el.disabled && el.offsetParent !== null);
        }
        function enableFocusTrap(modal) {
            const focusables = getFocusable(modal);
            if (!focusables.length) return;
            const first = focusables[0], last = focusables[focusables.length - 1];
            trapHandler = function(e) {
                if (e.key !== 'Tab') return;
                if (e.shiftKey && document.activeElement === first) { e.preventDefault(); last.focus(); }
                else if (!e.shiftKey && document.activeElement === last) { e.preventDefault(); first.focus(); }
            };
            modal.addEventListener('keydown', trapHandler);
        }
        function disableFocusTrap(modal) {
            if (trapHandler) modal.removeEventListener('keydown', trapHandler);
            trapHandler = null;
        }

        /* Refit on resize quando lightbox aberto */
        window.addEventListener('resize', () => {
            if (lightbox.classList.contains('active')) fitImageToStage();
        });
    </script>
</body>
</html>
