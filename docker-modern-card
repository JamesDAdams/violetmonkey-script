// ==UserScript==
// @name         Unraid Docker Modern Cards + Menu (dropdown compatible)
// @namespace    http://tampermonkey.net/
// @version      1.5
// @description  Affiche les containers Docker sous forme de cartes modernes avec menu contextuel compatible Unraid (dropdown natif).
// @author       JamesDAdams (Copilot)
// @match        https://192.168.1.100/Docker
// @grant        GM_addStyle
// ==/UserScript==

(function() {
    'use strict';

    // --- Modern Card + Menu style ---
    GM_addStyle(`
        .docker-card-list {
            display: flex;
            flex-wrap: wrap;
            gap: 24px;
            margin: 32px 0;
            justify-content: flex-start;
        }
        .docker-card {
            background: #181c21;
            color: #e2e8f0;
            border-radius: 16px;
            box-shadow: 0 4px 16px #0003;
            min-width: 360px;
            max-width: 390px;
            flex: 1 1 360px;
            padding: 24px 24px 18px 24px;
            display: flex;
            flex-direction: column;
            position: relative;
            font-family: 'Inter', 'Segoe UI', Arial, sans-serif;
            border: 2px solid #253251;
            transition: border-color 0.2s;
        }
        .docker-card[data-status="up-to-date"] { border-color: #426f46; }
        .docker-card[data-status="not available"] { border-color: #a58c2b; }
        .docker-card[data-status="other"] { border-color: #343a40; }

        .docker-card .card-header {
            display: flex;
            align-items: center;
            margin-bottom: 8px;
            position: relative;
        }
        .docker-card .app-logo {
            width: 36px;
            height: 36px;
            border-radius: 7px;
            object-fit: contain;
            margin-right: 16px;
            background: #23272e;
            border: 1px solid #23272e;
        }
        .docker-card .card-title {
            font-size: 1.22em;
            font-weight: bold;
            margin-right: 8px;
            color: #fff;
            margin-top: 2px;
        }
        .docker-card .badge {
            font-size: 0.98em;
            font-weight: 500;
            border-radius: 6px;
            padding: 3px 12px;
            margin-left: auto;
            background: #253251;
            color: #e2e8f0;
        }
        .docker-card[data-status="up-to-date"] .badge { background: #294e33; color: #b8e6b4;}
        .docker-card[data-status="not available"] .badge { background: #665c1a; color: #ffe680;}
        .docker-card[data-status="other"] .badge { background: #343a40; color: #e2e8f0;}

        .docker-card .docker-card-menu-btn {
            background: none;
            border: none;
            color: #b6c2e0;
            font-size: 1.5em;
            cursor: pointer;
            margin-left: 10px;
            margin-top: -5px;
            z-index: 2;
            transition: color 0.2s;
        }
        .docker-card .docker-card-menu-btn:hover { color: #fff; }

        /* Dropdown Unraid style override for visibility */
        .docker-card .dropdown-menu.dropdown-context.fa-ul {
            min-width: 210px;
            background: #23272e;
            border-radius: 10px;
            box-shadow: 0 6px 20px #0006;
            border: 1.5px solid #253251;
            z-index: 50;
            padding: 7px 0;
            font-size: 1.08em;
            position: absolute;
            left: 45px;
            top: 44px;
            display: none;
        }
        .docker-card .dropdown-menu li a {
            color: #e2e8f0;
            padding: 7px 20px;
            display: flex;
            align-items: center;
            font-size: 1em;
            border-bottom: 1px solid #253251;
            text-decoration: none;
            cursor: pointer;
            background: none;
            width: 100%;
        }
        .docker-card .dropdown-menu li:last-child a { border-bottom: none; }
        .docker-card .dropdown-menu li a:hover { background: #253251; color: #fff; }
        .docker-card .dropdown-menu .divider { border-bottom: 1.5px solid #23272e; margin: 4px 0;}
        .docker-card .dropdown-menu .fa-fw { margin-right: 12px; }
        .docker-card .dropdown-menu li a[target="_blank"] { color: #b8e6b4; }
        .docker-card .dropdown-menu li a[target="_blank"]:hover { color: #fff; }

        .docker-card .card-state {
            font-size: 1.05em;
            color: #50fa7b;
            display: flex;
            align-items: center;
            margin-bottom: 10px;
        }
        .docker-card .card-state .fa-play { color: #50fa7b; margin-right: 8px; }
        .docker-card[data-status="not available"] .card-state .fa-play { color: #ffe680 !important; }
        .docker-card .card-state .fa-stop { color: #ff4136; margin-right: 8px; }

        .docker-card .card-body {
            margin-bottom: 12px;
            font-size: 1.02em;
            line-height: 1.7;
        }
        .docker-card .card-body div {
            display: flex;
            justify-content: space-between;
            margin-bottom: 1px;
        }
        .docker-card .card-body strong { color: #b6c2e0; font-weight: 500;}
        .docker-card .card-sep {
            height: 1px;
            background: #253251;
            margin: 9px 0 8px 0;
            border-radius: 3px;
            opacity: 0.65;
        }
        .docker-card .card-footer {
            display: flex;
            flex-direction: column;
            gap: 2px;
            margin-top: 8px;
            font-size: 0.98em;
            color: #b6c2e0;
        }
        .docker-card .card-footer span {
            display: flex;
            justify-content: space-between;
        }
        .docker-card .autostart {
            font-weight: bold;
            color: #ff4136;
        }
        .docker-card[data-autostart="on"] .autostart { color: #4ae84a; }
    `);

    function normalizeActionKey(label) {
        return label ? label.replace(/\s+/g, ' ').trim().toLowerCase() : '';
    }

    // Dropdown native markup compatible Unraid
    function createDropdownMenu(container) {
        const ul = document.createElement('ul');
        ul.className = "dropdown-menu dropdown-context fa-ul";
        ul.style.display = "none";
        ul.id = "dropdown-" + container.containerId;

        const canRenderItem = (key, fallback) => {
            const link = container.actions?.[key];
            return !!link || !!(fallback && fallback.href);
        };

        function createMenuItem(icon, label, actionKey, fallback) {
            const actionLink = container.actions?.[actionKey];
            if (!actionLink && !(fallback && fallback.href)) return null;
            const li = document.createElement('li');
            const a = document.createElement('a');
            const href = actionLink?.href || fallback?.href || '#';
            const target = actionLink?.target || fallback?.target;
            a.tabIndex = -1;
            a.href = href;
            a.innerHTML = `<i class="fa fa-fw ${icon} fa-lg"></i>&nbsp;&nbsp;${label}`;
            a.addEventListener('click', function(e){
                e.preventDefault();
                e.stopPropagation();
                ul.style.display = "none";
                if (target === '_blank' && href && href !== '#') {
                    window.open(href, target);
                    return;
                }
                if (actionLink) {
                    actionLink.dispatchEvent(new MouseEvent('click', { bubbles: true, cancelable: true }));
                } else if (href && href !== '#') {
                    window.open(href, fallback?.target || '_self');
                }
            });
            li.appendChild(a);
            return li;
        }

        function divider() {
            const li = document.createElement('li');
            li.className = "divider";
            return li;
        }

        const groups = [
            [
                { icon: "fa-globe", label: "WebUI", key: "webui", fallback: { href: container.webuiUrl, target: "_blank" } },
                { icon: "fa-terminal", label: "Console", key: "console" }
            ],
            [
                { icon: "fa-play", label: "Start", key: "start" },
                { icon: "fa-play-circle", label: "Resume", key: "resume" },
                { icon: "fa-step-forward", label: "Unpause", key: "unpause" },
                { icon: "fa-stop", label: "Stop", key: "stop" },
                { icon: "fa-pause", label: "Pause", key: "pause" },
                { icon: "fa-refresh", label: "Restart", key: "restart" }
            ],
            [
                { icon: "fa-arrow-up", label: "Update", key: "update" },
                { icon: "fa-bolt", label: "Force Update", key: "force update" },
                { icon: "fa-check-circle", label: "Check for Updates", key: "check for updates" }
            ],
            [
                { icon: "fa-navicon", label: "Logs", key: "logs" },
                { icon: "fa-wrench", label: "Edit", key: "edit" },
                { icon: "fa-trash", label: "Remove", key: "remove" }
            ],
            [
                { icon: "fa-life-ring", label: "Project Page", key: "project page", fallback: { href: container.projectUrl, target: "_blank" } },
                { icon: "fa-question", label: "Support", key: "support", fallback: { href: container.supportUrl, target: "_blank" } },
                { icon: "fa-info-circle", label: "More Info", key: "more info", fallback: { href: container.projectUrl || container.supportUrl, target: "_blank" } },
                { icon: "fa-external-link", label: "Donate", key: "donate", fallback: { href: container.donateUrl, target: "_blank" } }
            ]
        ];

        const groupHasItems = groups.map(group => group.some(item => canRenderItem(item.key, item.fallback)));

        groups.forEach((group, index) => {
            if (!groupHasItems[index]) return;
            group.forEach(item => {
                const node = createMenuItem(item.icon, item.label, item.key, item.fallback);
                if (node) ul.appendChild(node);
            });
            if (groupHasItems.slice(index + 1).some(Boolean)) {
                ul.appendChild(divider());
            }
        });

        return ul;
    }

        function parseContainerRow(row) {
        const columns = row.querySelectorAll('td')
        if (columns.length < 10) return null


        let cpu = ''
        let ram = ''

        const appCell = columns[0]
        const img = appCell.querySelector('img')
        const logo = img ? img.src : ''
        const nameLink = appCell.querySelector('.appname a')
        const name = nameLink ? nameLink.textContent.trim() : ''
        const stateText = appCell.querySelector('.state')
        const state = stateText ? stateText.textContent.trim() : ''

        let updateText = 'other'
        if (columns[1].querySelector('span')) {
            updateText = columns[1].querySelector('span').textContent.trim().toLowerCase()
        }

        const network = columns[2].textContent.trim()
        const ip = columns[3].textContent.trim()
        const port = columns[4].textContent.trim()

        const outerSpan = columns[0].querySelector('.outer > span')
        const containerId = outerSpan ? outerSpan.id : ''

        const cpuSpan = document.querySelector(`span.cpu-${containerId}`)
        const ramSpan = document.querySelector(`span.mem-${containerId}`)

        if (cpuSpan) cpu = cpuSpan.textContent.trim()
        if (ramSpan) ram = ramSpan.textContent.trim()

        if (cpuSpan && ramSpan) {
            const observer = new MutationObserver(() => {
                cpu = cpuSpan.textContent.trim()
                ram = ramSpan.textContent.trim()
                const card = document.querySelector(`.docker-card[data-id="${containerId}"]`)
                if (card) {
                    const cpuField = card.querySelector('.card-body div:nth-child(5) span')
                    const ramField = card.querySelector('.card-body div:nth-child(6) span')
                    if (cpuField) cpuField.textContent = cpu
                    if (ramField) ramField.textContent = ram
                }
            })
            observer.observe(cpuSpan, { childList: true })
            observer.observe(ramSpan, { childList: true })
        } else {
            console.warn(`Spans non trouvés pour containerId ${containerId}`)
        }

        let autostart = false
        const autostartOn = row.querySelector('.switch-button-label.on')
        if (autostartOn && autostartOn.style.display !== 'none') {
            autostart = true
        }

        let uptime = ''
        let created = ''
        const uptimeDiv = columns[9].querySelector('div')
        if (uptimeDiv) {
            const text = uptimeDiv.textContent.replace(/\s+/g, ' ').trim()
            const upMatch = text.match(/Uptime:\s*([^C]*)/i)
            const createdMatch = text.match(/Created:\s*([^\n]*)/i)
            uptime = upMatch ? upMatch[1].trim() : ''
            created = createdMatch ? createdMatch[1].trim() : ''
        }

        let webuiUrl = "", projectUrl = "", supportUrl = "", donateUrl = ""
        if (outerSpan && outerSpan.hasAttribute('onclick')) {
            const argStr = outerSpan.getAttribute('onclick')
            const args = argStr.split("','")
            if (args.length > 8) {
                webuiUrl = args[7].replace(/'/g, "")
                projectUrl = args[12] ? args[12].replace(/'/g, "") : ""
                supportUrl = args[13] ? args[13].replace(/'/g, "") : ""
                donateUrl = args[15] ? args[15].replace(/'/g, "") : ""
            }
        }

        const actions = {}
        columns[0].querySelectorAll('.dropdown-menu.dropdown-context.fa-ul li a').forEach(anchor => {
            const key = normalizeActionKey(anchor.textContent || anchor.innerText)
            if (key) actions[key] = anchor
        })

        return {
            name,
            logo,
            state,
            updateText,
            network,
            ip,
            port,
            cpu,
            ram,
            autostart,
            uptime,
            created,
            webuiUrl,
            projectUrl,
            supportUrl,
            donateUrl,
            containerId,
            actions
        }


        }


    function createCard(container) {
        let status = "other";
        if (container.updateText.includes("up-to-date")) status = "up-to-date";
        else if (container.updateText.includes("not available")) status = "not available";

        let stateIcon = '<i class="fa fa-play"></i>';
        let stateText = "Running";
        if (container.state.toLowerCase().includes("started")) {
            stateIcon = '<i class="fa fa-play"></i>';
            stateText = "Running";
        } else {
            stateIcon = '<i class="fa fa-stop"></i>';
            stateText = "Stopped";
        }

        const autoText = container.autostart ? "ON" : "OFF";

        const card = document.createElement('div');
        card.className = 'docker-card';
        card.dataset.status = status
        card.dataset.autostart = container.autostart ? 'on' : 'off'
        card.dataset.id = container.containerId
        card.setAttribute('data-status', status);
        card.setAttribute('data-autostart', container.autostart ? 'on' : 'off');

        // --- Injection du span original avec onclick ---
        // On récupère le span original depuis le tableau source
        const originalSpan = document.getElementById(container.containerId);
        let spanHtml = "";
        let spanOnClick = "";
        if (originalSpan) {
            // On clone le span pour éviter de le retirer du DOM original
            const clonedSpan = originalSpan.cloneNode(true);
            spanHtml = clonedSpan.outerHTML;
            if (clonedSpan.getAttribute('onclick')) {
                spanOnClick = clonedSpan.getAttribute('onclick');
            }
        }

        const headerDiv = document.createElement('div');
        headerDiv.className = 'card-header';
        headerDiv.innerHTML = `
            ${spanHtml}
            <span class="card-title">${container.name}</span>
            <span class="badge">${container.updateText}</span>
        `;

        // Suppression du bouton menuBtn

        // Ajout du menu contextuel
        const dropdownMenu = createDropdownMenu(container);
        headerDiv.appendChild(dropdownMenu);

        // Affichage du menu au clic sur le bouton
        // menuBtn supprimé, donc pas d'événement à ajouter ici

        card.appendChild(headerDiv);

        const stateDiv = document.createElement('div');
        stateDiv.className = 'card-state';
        stateDiv.innerHTML = `${stateIcon} ${stateText}`;
        card.appendChild(stateDiv);

        const bodyDiv = document.createElement('div');
        bodyDiv.className = 'card-body';
        bodyDiv.innerHTML = `
            <div><strong>Network:</strong> <span>${container.network}</span></div>
            <div><strong>IP:</strong> <span>${container.ip}</span></div>
            <div><strong>Port:</strong> <span>${container.port}</span></div>
            <div class="card-sep"></div>
            ${
                isAdvancedViewEnabled()
                ? `<div><strong>CPU:</strong> <span>${container.cpu}</span></div>
                   <div><strong>RAM:</strong> <span>${container.ram}</span></div>`
                : ''
            }
        `;
        card.appendChild(bodyDiv);

        const footerDiv = document.createElement('div');
        footerDiv.className = 'card-footer';
        // Correction : chaque info sur une ligne séparée
        footerDiv.innerHTML = `
            <span><strong>Uptime:</strong> <span>${container.uptime}</span></span>
            <span><strong>Created:</strong> <span>${container.created}</span></span>
            <span><strong>Autostart:</strong> <span class="autostart">${autoText}</span></span>
        `;
        card.appendChild(footerDiv);

        return card;
    }

    function replaceTableWithCards() {
        const table = document.getElementById('docker_containers');
        if (!table) return;

        const tbody = document.getElementById('docker_list') || table.querySelector('tbody');
        if (!tbody) return;

        const previousCards = document.querySelector('.docker-card-list');
        if (previousCards) previousCards.remove();

        const rows = Array.from(tbody.querySelectorAll('tr.sortable'));
        const containers = rows.map(parseContainerRow).filter(Boolean);

        if (!containers.length) {
            table.style.display = '';
            return;
        }

        const cardList = document.createElement('div');
        cardList.className = 'docker-card-list';

        containers.forEach(container => {
            cardList.appendChild(createCard(container));
        });

        table.parentNode.insertBefore(cardList, table.nextSibling);
        table.style.display = 'none';
    }

    function observeTable() {
        const tbody = document.getElementById('docker_list') || document.querySelector('#docker_containers tbody');
        if (!tbody || window.__dockerCardsObserver) return;

        const observer = new MutationObserver(() => replaceTableWithCards());
        observer.observe(tbody, { childList: true, subtree: false });
        window.__dockerCardsObserver = observer;
    }

    // Vérifie si la vue avancée est activée
    function isAdvancedViewEnabled() {
        const checkbox = document.querySelector('input.advancedview');
        if (checkbox) return checkbox.checked;
        // Fallback: label "Advanced View" visible
        const labelOn = document.querySelector('.switch-button-label.on');
        return labelOn && labelOn.style.display !== 'none';
    }

    function observeAdvancedViewSwitch() {
        const checkbox = document.querySelector('input.advancedview');
        if (!checkbox || window.__dockerCardsAdvancedViewObs) return;
        checkbox.addEventListener('change', () => {
            replaceTableWithCards();
        });
        window.__dockerCardsAdvancedViewObs = true;
    }

    window.addEventListener('DOMContentLoaded', function() {
        replaceTableWithCards();
        observeTable();
        observeAdvancedViewSwitch();
    });
    setTimeout(function() {
        replaceTableWithCards();
        observeTable();
        observeAdvancedViewSwitch();
    }, 1500);

})();
