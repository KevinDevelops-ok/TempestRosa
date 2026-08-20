/* ==========================================================================
   TEMPEST ROSA — script.js
   Vanilla JS. No frameworks, no dependencies.
   ========================================================================== */

(function () {
  'use strict';

  var reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

  /* ------------------------------------------------------------------
     ENTRY EXPERIENCE
  ------------------------------------------------------------------ */
  function initEntry() {
    var entry = document.getElementById('entry');
    var enterBtn = document.getElementById('enterBtn');
    var site = document.getElementById('site');
    if (!entry || !enterBtn || !site) return;

    function enter() {
      entry.classList.add('is-leaving');
      entry.setAttribute('aria-hidden', 'true');
      site.setAttribute('aria-hidden', 'false');

      // reveal the site behind the entry as it fades
      requestAnimationFrame(function () {
        site.classList.add('is-revealed');
      });

      var cleanup = function () {
        entry.style.display = 'none';
        enterBtn.removeEventListener('click', enter);
        document.body.classList.remove('has-entry');
      };

      if (reduceMotion) {
        cleanup();
      } else {
        entry.addEventListener('transitionend', cleanup, { once: true });
        // safety fallback in case transitionend doesn't fire
        setTimeout(cleanup, 1400);
      }
    }

    document.body.classList.add('has-entry');
    enterBtn.addEventListener('click', enter);

    // Allow pressing Enter/Space to cross the threshold from anywhere
    entry.addEventListener('keydown', function (e) {
      if (e.key === 'Enter' || e.key === ' ') {
        e.preventDefault();
        enter();
      }
    });
  }

  /* ------------------------------------------------------------------
     STICKY NAV — shadow/border once scrolled
  ------------------------------------------------------------------ */
  function initNavScrollState() {
    var nav = document.getElementById('siteNav');
    if (!nav) return;
    var ticking = false;

    function update() {
      nav.classList.toggle('is-scrolled', window.scrollY > 8);
      ticking = false;
    }
    window.addEventListener('scroll', function () {
      if (!ticking) {
        requestAnimationFrame(update);
        ticking = true;
      }
    }, { passive: true });
    update();
  }

  /* ------------------------------------------------------------------
     MOBILE MENU — full-screen editorial overlay
  ------------------------------------------------------------------ */
  function initMobileMenu() {
    var toggle = document.getElementById('menuToggle');
    var closeBtn = document.getElementById('menuClose');
    var menu = document.getElementById('mobileMenu');
    if (!toggle || !menu || !closeBtn) return;

    var lastFocused = null;

    function openMenu() {
      lastFocused = document.activeElement;
      menu.classList.add('is-open');
      menu.setAttribute('aria-hidden', 'false');
      toggle.setAttribute('aria-expanded', 'true');
      document.body.classList.add('menu-open');
      closeBtn.focus();
      document.addEventListener('keydown', onKeydown);
    }

    function closeMenu() {
      menu.classList.remove('is-open');
      menu.setAttribute('aria-hidden', 'true');
      toggle.setAttribute('aria-expanded', 'false');
      document.body.classList.remove('menu-open');
      document.removeEventListener('keydown', onKeydown);
      if (lastFocused) lastFocused.focus();
    }

    function onKeydown(e) {
      if (e.key === 'Escape') {
        closeMenu();
        return;
      }
      if (e.key === 'Tab') {
        var focusable = menu.querySelectorAll('a, button');
        var first = focusable[0];
        var last = focusable[focusable.length - 1];
        if (e.shiftKey && document.activeElement === first) {
          e.preventDefault();
          last.focus();
        } else if (!e.shiftKey && document.activeElement === last) {
          e.preventDefault();
          first.focus();
        }
      }
    }

    toggle.addEventListener('click', openMenu);
    closeBtn.addEventListener('click', closeMenu);
    menu.querySelectorAll('a').forEach(function (link) {
      link.addEventListener('click', closeMenu);
    });
  }

  /* ------------------------------------------------------------------
     SCROLL REVEAL — IntersectionObserver, cheap and one-shot
  ------------------------------------------------------------------ */
  function initScrollReveal() {
    var items = document.querySelectorAll('[data-reveal]');
    if (!items.length) return;

    if (reduceMotion || !('IntersectionObserver' in window)) {
      items.forEach(function (el) { el.classList.add('is-visible'); });
      return;
    }

    var observer = new IntersectionObserver(function (entries) {
      entries.forEach(function (entry, i) {
        if (entry.isIntersecting) {
          var el = entry.target;
          var delay = Math.min(i * 40, 160);
          setTimeout(function () { el.classList.add('is-visible'); }, delay);
          observer.unobserve(el);
        }
      });
    }, { threshold: 0.15, rootMargin: '0px 0px -60px 0px' });

    items.forEach(function (el) { observer.observe(el); });
  }

  /* ------------------------------------------------------------------
     ORBIT — pause the hero signature animation when off-screen
  ------------------------------------------------------------------ */
  function initOrbit() {
    var orbit = document.querySelector('[data-orbit]');
    if (!orbit || reduceMotion || !('IntersectionObserver' in window)) return;

    var observer = new IntersectionObserver(function (entries) {
      entries.forEach(function (entry) {
        entry.target.classList.toggle('is-active', entry.isIntersecting);
      });
    }, { threshold: 0.1 });

    observer.observe(orbit);
  }

  /* ------------------------------------------------------------------
     PROJECT INTAKE — option select reveals form (Community section)
  ------------------------------------------------------------------ */
  function initIntake() {
    var options = document.querySelectorAll('.intake__opt');
    var form = document.getElementById('intakeForm');
    var choiceLabel = document.getElementById('intakeChoice');
    var categoryField = document.getElementById('f-category');
    var whatField = document.getElementById('f-what');
    var successMsg = document.getElementById('intakeSuccess');
    var errorMsg = document.getElementById('intakeError');
    var submitBtn = form ? form.querySelector('.intake__submit') : null;
    var submitLabel = form ? form.querySelector('.intake__submit-label') : null;
    if (!options.length || !form) return;

    options.forEach(function (btn) {
      btn.addEventListener('click', function () {
        options.forEach(function (b) { b.classList.remove('is-selected'); b.setAttribute('aria-pressed', 'false'); });
        btn.classList.add('is-selected');
        btn.setAttribute('aria-pressed', 'true');

        var value = btn.getAttribute('data-value');
        if (choiceLabel) choiceLabel.textContent = value.toLowerCase();
        if (categoryField) categoryField.value = value;
        if (whatField && !whatField.value) whatField.value = value === 'Not sure yet' ? '' : value;

        if (form.hasAttribute('hidden')) {
          form.hidden = false;
          var firstField = document.getElementById('f-name');
          if (firstField) firstField.focus();
        }
      });
    });

    /*
      Validation — runs before any send attempt. Catches an empty name
      or a contact field that isn't recognisable as an email or phone
      number, right at the point of entry, with an inline message next
      to the field itself. This is separate from (and runs before) the
      actual send — see the EmailJS block below for what happens if the
      email genuinely fails to go through after validation passes.
    */
    var nameField = document.getElementById('f-name');
    var contactField = document.getElementById('f-contact');
    var nameError = document.getElementById('f-name-error');
    var contactError = document.getElementById('f-contact-error');

    var EMAIL_RE = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    var PHONE_RE = /^[+]?[\d\s().-]{7,}$/;

    function setFieldError(input, errorEl, message) {
      if (!input) return;
      input.closest('.field').classList.add('has-error');
      input.setAttribute('aria-invalid', 'true');
      if (errorEl) {
        if (message) errorEl.textContent = message;
        errorEl.hidden = false;
      }
    }

    function clearFieldError(input, errorEl) {
      if (!input) return;
      input.closest('.field').classList.remove('has-error');
      input.removeAttribute('aria-invalid');
      if (errorEl) errorEl.hidden = true;
    }

    function validateName() {
      var value = nameField.value.trim();
      if (!value) {
        setFieldError(nameField, nameError);
        return false;
      }
      clearFieldError(nameField, nameError);
      return true;
    }

    function validateContact() {
      var value = contactField.value.trim();
      var looksValid = EMAIL_RE.test(value) || PHONE_RE.test(value);
      if (!value || !looksValid) {
        setFieldError(contactField, contactError);
        return false;
      }
      clearFieldError(contactField, contactError);
      return true;
    }

    // Validate as the visitor leaves each field, not just on submit —
    // catches typos immediately instead of after they hit send.
    if (nameField) nameField.addEventListener('blur', validateName);
    if (contactField) contactField.addEventListener('blur', validateContact);

    /*
      Real submission via EmailJS — see the HTML comment above the
      submit button, and README.md, for the one-time setup step
      (writing your own email template in the EmailJS dashboard, then
      pasting your Service ID / Template ID / Public Key onto #intakeForm).

      Until those are set, submissions are logged to the console
      instead of silently "succeeding", so it's obvious during
      development that setup isn't finished yet.
    */
    form.addEventListener('submit', function (e) {
      e.preventDefault();
      if (errorMsg) errorMsg.hidden = true;

      var nameOk = validateName();
      var contactOk = validateContact();
      if (!nameOk || !contactOk) {
        (nameOk ? contactField : nameField).focus();
        return;
      }

      // Honeypot: if this hidden field got filled in, it was a bot —
      // pretend to succeed without actually sending anything.
      var honeypot = form.querySelector('[name="_gotcha"]');
      if (honeypot && honeypot.value) {
        showSuccess();
        return;
      }

      var serviceId = form.getAttribute('data-emailjs-service-id');
      var templateId = form.getAttribute('data-emailjs-template-id');
      var publicKey = form.getAttribute('data-emailjs-public-key');
      var isConfigured = serviceId && serviceId !== 'YOUR_SERVICE_ID' &&
        templateId && templateId !== 'YOUR_TEMPLATE_ID' &&
        publicKey && publicKey !== 'YOUR_PUBLIC_KEY';

      function setLoading(isLoading) {
        if (submitBtn) submitBtn.disabled = isLoading;
        if (submitLabel) submitLabel.textContent = isLoading ? 'SENDING…' : 'BEGIN A PROJECT →';
      }

      function showSuccess() {
        form.querySelectorAll('input, textarea, select, button').forEach(function (el) {
          el.disabled = true;
        });
        if (successMsg) successMsg.hidden = false;
      }

      function showError() {
        setLoading(false);
        if (errorMsg) errorMsg.hidden = false;
      }

      if (!isConfigured) {
        // Setup not finished yet — make that visible in the console
        // rather than pretending the message was sent.
        console.warn(
          'Tempest Rosa intake form: EmailJS is not configured yet. ' +
          'Set data-emailjs-service-id, data-emailjs-template-id and ' +
          'data-emailjs-public-key on #intakeForm to start receiving ' +
          'real submissions by email, written in your own words. ' +
          'See README.md for the setup steps.'
        );
        setLoading(true);
        setTimeout(function () {
          setLoading(false);
          showSuccess();
        }, 500);
        return;
      }

      if (typeof emailjs === 'undefined') {
        console.error('Tempest Rosa intake form: the EmailJS SDK failed to load (offline, or blocked).');
        showError();
        return;
      }

      // Merge fields available in your EmailJS template:
      // {{name}} {{contact}} {{what}} {{description}} {{references}} {{budget}} {{category}}
      var templateParams = {
        name: form.querySelector('#f-name').value,
        contact: form.querySelector('#f-contact').value,
        what: form.querySelector('#f-what').value,
        description: form.querySelector('#f-desc').value,
        references: form.querySelector('#f-refs').value,
        budget: form.querySelector('#f-budget').value || 'Not specified',
        category: form.querySelector('#f-category').value || 'Not specified'
      };

      setLoading(true);
      emailjs.send(serviceId, templateId, templateParams, { publicKey: publicKey })
        .then(function () {
          setLoading(false);
          showSuccess();
        })
        .catch(function (err) {
          console.error('Tempest Rosa intake form: EmailJS send failed.', err);
          showError();
        });
    });
  }

  /* ------------------------------------------------------------------
     INIT
  ------------------------------------------------------------------ */
  document.addEventListener('DOMContentLoaded', function () {
    document.documentElement.classList.remove('no-js');
    initEntry();
    initNavScrollState();
    initMobileMenu();
    initScrollReveal();
    initOrbit();
    initIntake();
  });
})();
