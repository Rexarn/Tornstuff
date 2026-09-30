// ==UserScript==
// @name         Torn Item Market - Final Price After Fees - by Rexarn
// @namespace    http://tampermonkey.net/
// @version      1.4
// @description  Shows net FINAL PRICE after sale + anonymity fees beside the Show Popup checkbox
// @author       You
// @match        https://www.torn.com/page.php?sid=ItemMarket*
// @match        https://www.torn.com/imarket.php*
// @grant        none
// @run-at       document-idle
// ==/UserScript==

(function () {
    'use strict';

    const FINAL_COLOR = '#34c0eb';
    const MARKER_CLASS = 'tm-final-price-after-fees';

    function parseMoney(str) {
        if (!str) return 0;
        return parseInt(String(str).replace(/[$,\s]/g, ''), 10) || 0;
    }

    function formatMoney(num) {
        return '$' + num.toLocaleString('en-US');
    }

    function findAndInject() {
        const root = document.getElementById('item-market-root');
        if (!root) return;

        // Find the summary text that contains the fee info
        let total = 0, saleFee = 0, anonFee = 0;
        let foundFees = false;

        const candidates = root.querySelectorAll('div');
        for (const el of candidates) {
            const text = el.textContent || '';
            if (!/You are adding .+ to the Item Market/i.test(text)) continue;
            if (!/sale fee/i.test(text)) continue;

            const totalMatch   = text.match(/total of \$([\d,]+)/i);
            const saleFeeMatch = text.match(/sale fee.*?\$([\d,]+)/i);
            const anonFeeMatch = text.match(/anonymity fee.*?\$([\d,]+)/i);

            if (!totalMatch || !saleFeeMatch) continue;

            total   = parseMoney(totalMatch[1]);
            saleFee = parseMoney(saleFeeMatch[1]);
            anonFee = anonFeeMatch ? parseMoney(anonFeeMatch[1]) : 0;
            foundFees = true;
            break;
        }

        if (!foundFees) return;

        const final = total - saleFee - anonFee;

        // Target the Show Popup checkbox container
        const checkboxContainer = document.getElementById('disable-popup-checkbox-container');
        if (!checkboxContainer) return;

        // Remove any previous marker
        root.querySelectorAll(`.${MARKER_CLASS}`).forEach(n => n.remove());

        // Create the final price element
        const finalEl = document.createElement('span');
        finalEl.className = MARKER_CLASS;
        finalEl.style.cssText = `
            margin-left: 6px;
            font-weight: normal;
            font-size: 1.00em;
            color: ${FINAL_COLOR};
            white-space: nowrap;
            vertical-align: text-bottom;
        `;
        finalEl.textContent = `FINAL PRICE AFTER SALE: ${formatMoney(final)}`;

        // Insert right after the checkbox container (beside "Show Popup")
        checkboxContainer.insertAdjacentElement('afterend', finalEl);
    }

    // Observe the React root for changes
    const observer = new MutationObserver(() => {
        clearTimeout(window.__tmFeeTimer);
        window.__tmFeeTimer = setTimeout(findAndInject, 100);
    });

    function startObserving() {
        const root = document.getElementById('item-market-root');
        if (root) {
            observer.observe(root, {
                childList: true,
                subtree: true,
                characterData: true
            });
            findAndInject();
        } else {
            setTimeout(startObserving, 300);
        }
    }

    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', startObserving);
    } else {
        startObserving();
    }
})();
