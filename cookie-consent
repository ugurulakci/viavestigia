// Via Vestigia — shared cookie consent controller
// Loaded identically on all five pages (homepage, /experiences/, and the
// three product pages). This file is the ONLY place that can inject the
// GA4 and Microsoft Clarity scripts — neither script tag lives in any
// page's own <head> anymore. Nothing loads, and nothing is sent to
// Google or Microsoft, until the visitor makes an explicit choice.

(function () {
  var STORAGE_KEY = 'vv_analytics_consent'; // 'accepted' | 'rejected'
  var CLARITY_ID = 'y5qrv95a45';
  var GA_ID = 'G-KWJCM7ZM31';

  function getConsent() {
    try { return localStorage.getItem(STORAGE_KEY); } catch (e) { return null; }
  }
  function setConsent(value) {
    try { localStorage.setItem(STORAGE_KEY, value); } catch (e) {}
  }

  function loadGA4() {
    if (window.__vvGA4Loaded) return;
    window.__vvGA4Loaded = true;
    var s = document.createElement('script');
    s.async = true;
    s.src = 'https://www.googletagmanager.com/gtag/js?id=' + GA_ID;
    document.head.appendChild(s);
    window.dataLayer = window.dataLayer || [];
    window.gtag = window.gtag || function () { window.dataLayer.push(arguments); };
    gtag('js', new Date());
    gtag('config', GA_ID);
  }

  function loadClarity() {
    if (window.__vvClarityLoaded) return;
    window.__vvClarityLoaded = true;
    (function (c, l, a, r, i, t, y) {
      c[a] = c[a] || function () { (c[a].q = c[a].q || []).push(arguments); };
      t = l.createElement(r); t.async = 1; t.src = "https://www.clarity.ms/tag/" + i;
      y = l.getElementsByTagName(r)[0]; y.parentNode.insertBefore(t, y);
    })(window, document, "clarity", "script", CLARITY_ID);
    // Ads consent is always denied — the site never uses ad/remarketing cookies.
    window.clarity('consentv2', { ad_Storage: 'denied', analytics_Storage: 'granted' });
  }

  function clearAnalyticsCookies() {
    // GA4: _ga, _ga_<id>, _gid, _gat  |  Clarity: _clck, _clsk — all first-party.
    var prefixes = ['_ga', '_gid', '_gat', '_clck', '_clsk'];
    document.cookie.split(';').forEach(function (c) {
      var name = c.split('=')[0].trim();
      if (prefixes.some(function (p) { return name.indexOf(p) === 0; })) {
        document.cookie = name + '=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;';
      }
    });
    if (window.clarity) {
      window.clarity('consentv2', { ad_Storage: 'denied', analytics_Storage: 'denied' });
    }
  }

  function showBanner() {
    var el = document.getElementById('vvCookieBanner');
    if (el) el.hidden = false;
  }
  function hideBanner() {
    var el = document.getElementById('vvCookieBanner');
    if (el) el.hidden = true;
  }

  function init() {
    var consent = getConsent();

    if (consent === 'accepted') {
      loadGA4();
      loadClarity();
      hideBanner();
    } else if (consent === 'rejected') {
      hideBanner();
    } else {
      showBanner();
    }

    var acceptBtn = document.getElementById('vvCookieAccept');
    var rejectBtn = document.getElementById('vvCookieReject');
    // "Cookie Preferences": a control that REOPENS this banner — not a normal
    // navigation link. Kept structurally distinct from the Privacy & Cookie
    // Policy link, which stays a plain <a> to the policy page.
    var prefsBtn = document.getElementById('vvCookiePrefsLink');

    if (acceptBtn) {
      acceptBtn.addEventListener('click', function () {
        var wasAlreadyLoaded = window.__vvGA4Loaded || window.__vvClarityLoaded;
        setConsent('accepted');
        if (wasAlreadyLoaded) {
          // Re-opened via Cookie Preferences after a prior Reject: scripts were
          // never loaded in THIS state, so a plain load is enough — nothing to
          // reconcile. (This branch only matters if a future change adds a
          // "reject after accept without loaded scripts" edge case; currently
          // reject never loads anything, so this is effectively unreachable —
          // kept for clarity rather than removed.)
        }
        loadGA4();
        loadClarity();
        hideBanner();
      });
    }
    if (rejectBtn) {
      rejectBtn.addEventListener('click', function () {
        var wasLoaded = window.__vvGA4Loaded || window.__vvClarityLoaded;
        setConsent('rejected');
        if (wasLoaded) {
          // Switching Accept -> Reject via Cookie Preferences: clear what's
          // there now, then reload so init() runs fresh and never re-injects
          // either script. This is more reliable than trying to partially
          // silence an already-running gtag.js in place.
          clearAnalyticsCookies();
          location.reload();
        } else {
          hideBanner();
        }
      });
    }
    if (prefsBtn) {
      prefsBtn.addEventListener('click', function (e) {
        e.preventDefault();
        showBanner();
      });
    }
  }

  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', init);
  } else {
    init();
  }
})();
