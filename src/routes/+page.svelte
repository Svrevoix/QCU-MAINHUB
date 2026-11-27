<script lang="ts">
	import { onMount } from 'svelte';

	type Theme = 'gold' | 'royal' | 'sky';

	let theme = $state<Theme>('gold');
	let tick = $state(0);
	let topCard = $state(0);
	let leaving = $state(false);
	let startX = 0;
	let currentDayIndex = $state<number | null>(null);
	let swipeTimer: ReturnType<typeof setTimeout> | undefined;

	const weekdays = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
	const themeOrder: Theme[] = ['gold', 'royal', 'sky'];
	const themeNames: Record<Theme, string> = {
		gold: 'QCU Navy & Gold',
		royal: 'Midnight Royal',
		sky: 'Campus Sky Mist'
	};
	const swatches: Record<Theme, string> = {
		gold: 'linear-gradient(135deg,#0b1330,#e9b52c)',
		royal: 'linear-gradient(135deg,#0b1440,#22d3ee)',
		sky: 'linear-gradient(135deg,#bfdbfe,#ffffff)'
	};
	const words = {
		kicker: 'Quezon City University \u2022 Consumers Cooperative',
		who: 'Who we are?',
		mission: 'Mission',
		vision: 'Vision',
		commit: 'Our commitment',
		values: 'Values',
		offer: 'What we offer?',
		purpose: 'Our purpose',
		why: 'Why a coop in the university?',
		how1: 'How it works:',
		portal: 'Members Portal',
		online: 'Your coop, online',
		member: 'Membership',
		reqs: 'Requirements:',
		faq: 'Common questions',
		store: 'Store Portal',
		receipt: 'Print membership \u00b7 Official receipt',
		services: 'Printing Services',
		how2: 'How it works?',
		promos: 'Printing Promos',
		wallet: 'Wallet \u00b7 Print & Earn Rewards',
		intro: 'Introducing',
		about: 'About',
		find: 'Find us!',
		explore: 'Explore',
		portals: 'Portals',
		contact: 'Contact'
	};
	const digits = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
	const typed = (text: string, speed: number) => {
		const period = Math.ceil(text.length / speed) + 90;
		const index = Math.floor((tick % period) * speed);
		return text.slice(0, index);
	};
	const themeName = $derived(themeNames[theme]);
	const swatch = $derived(swatches[theme]);
	const t = $derived({
		kicker: typed(words.kicker, 0.6),
		who: typed(words.who, 0.6),
		mission: typed(words.mission, 0.6),
		vision: typed(words.vision, 0.6),
		commit: typed(words.commit, 0.6),
		values: typed(words.values, 0.6),
		offer: typed(words.offer, 0.6),
		purpose: typed(words.purpose, 0.6),
		why: typed(words.why, 0.6),
		how1: typed(words.how1, 0.6),
		portal: typed(words.portal, 0.6),
		online: typed(words.online, 0.6),
		member: typed(words.member, 0.6),
		reqs: typed(words.reqs, 0.6),
		faq: typed(words.faq, 0.6),
		store: typed(words.store, 0.6),
		receipt: typed(words.receipt, 0.6),
		services: typed(words.services, 0.6),
		how2: typed(words.how2, 0.6),
		promos: typed(words.promos, 0.6),
		wallet: typed(words.wallet, 0.6),
		intro: typed(words.intro, 0.6),
		about: typed(words.about, 0.6),
		find: typed(words.find, 0.6),
		explore: typed(words.explore, 0.6),
		portals: typed(words.portals, 0.6),
		contact: typed(words.contact, 0.6)
	});
	const heroText = $derived(typed('By the campus. For the campus.', 0.8));
	const joinProgress = $derived(tick % 110);
	const joinText = $derived('Five steps to joining'.slice(0, Math.floor(joinProgress * 0.6)));
	const joinFade = $derived(joinProgress > 100 ? 0 : 1);
	const rollActive = $derived(tick % 90 > 3);
	const roll = (n: number) => (rollActive ? -56 * n : 0);
	const d1 = $derived(roll(2));
	const d2 = $derived(roll(1));
	const d3 = $derived(roll(6));
	const l1 = $derived(roll(4));
	const five = $derived(roll(5));
	const currentDayLabel = $derived(currentDayIndex === null ? '—' : weekdays[currentDayIndex]);
	const isWeekday = $derived(
		currentDayIndex !== null && currentDayIndex >= 1 && currentDayIndex <= 5
	);
	const card = (i: number) => {
		const position = (i - topCard + 3) % 3;
		const out = position === 0 && leaving;
		return {
			z: 3 - position,
			o: out ? 0 : 1,
			t: out
				? 'translateX(-120%) rotate(-18deg)'
				: `translate(${position * 12}px, ${position * 15}px) rotate(${position * 3}deg)`
		};
	};
	const c0 = $derived(card(0));
	const c1 = $derived(card(1));
	const c2 = $derived(card(2));

	function nextTheme() {
		theme = themeOrder[(themeOrder.indexOf(theme) + 1) % themeOrder.length];
	}

	function swipe() {
		if (leaving) return;
		leaving = true;
		swipeTimer = setTimeout(() => {
			topCard = (topCard + 1) % 3;
			leaving = false;
		}, 380);
	}

	function down(event: PointerEvent) {
		startX = event.clientX;
		const card = event.target instanceof Element ? event.target.closest('button.photo') : null;
		card?.setPointerCapture(event.pointerId);
	}

	function up(event: PointerEvent) {
		if (Math.abs(event.clientX - startX) > 40) swipe();
	}

	function headingTypewriter(node: HTMLElement) {
		if (node.querySelector('.type')) return;

		const walker = document.createTreeWalker(node, NodeFilter.SHOW_TEXT);
		const textNodes: Text[] = [];
		while (walker.nextNode()) {
			if (walker.currentNode instanceof Text) textNodes.push(walker.currentNode);
		}

		const text = textNodes.map((textNode) => textNode.nodeValue ?? '').join('');
		const characters = Array.from(text);
		const characterSpans: HTMLSpanElement[] = [];
		for (const textNode of textNodes) {
			const fragment = document.createDocumentFragment();
			for (const character of Array.from(textNode.nodeValue ?? '')) {
				const span = document.createElement('span');
				span.className = 'typewriter-char';
				span.textContent = character;
				characterSpans.push(span);
				fragment.appendChild(span);
			}
			textNode.parentNode?.replaceChild(fragment, textNode);
		}

		if (!characterSpans.length) return;

		if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return;

		characterSpans.forEach((span) => span.classList.remove('is-visible'));
		const caret = document.createElement('span');
		caret.className = 'caret';
		caret.setAttribute('aria-hidden', 'true');
		node.appendChild(caret);

		let observer: IntersectionObserver | undefined;
		let typewriterTimer: number | undefined;
		const startTyping = () => {
			if (typewriterTimer !== undefined) return;

			let sliceIndex = 0;
			let characterIndex = 0;
			typewriterTimer = window.setInterval(() => {
				const nextCharacter = characters[characterIndex];
				if (nextCharacter === undefined) {
					if (typewriterTimer !== undefined) window.clearInterval(typewriterTimer);
					typewriterTimer = undefined;
					return;
				}

				sliceIndex += nextCharacter.length;
				characterIndex += 1;
				const visibleText = text.slice(0, sliceIndex);
				const visibleCount = Array.from(visibleText).length;
				characterSpans[visibleCount - 1]?.classList.add('is-visible');

				if (characterIndex === characters.length) {
					if (typewriterTimer !== undefined) window.clearInterval(typewriterTimer);
					typewriterTimer = undefined;
				}
			}, 55);
		};
		if ('IntersectionObserver' in window) {
			observer = new IntersectionObserver((entries) => {
				if (!entries.some((entry) => entry.isIntersecting)) return;
				startTyping();
				observer?.disconnect();
			});
			observer.observe(node);
		} else {
			startTyping();
		}

		return {
			destroy: () => {
				observer?.disconnect();
				if (typewriterTimer !== undefined) window.clearInterval(typewriterTimer);
			}
		};
	}

	onMount(() => {
		const updateCurrentDay = () => {
			const today = new Date().getDay();
			if (currentDayIndex !== today) currentDayIndex = today;
		};
		updateCurrentDay();

		const timer = window.setInterval(() => tick++, 80);
		const dayTimer = window.setInterval(updateCurrentDay, 60_000);
		return () => {
			window.clearInterval(timer);
			window.clearInterval(dayTimer);
			if (swipeTimer !== undefined) window.clearTimeout(swipeTimer);
		};
	});
</script>

<svelte:head>
	<title>QCU Consumers Cooperative</title>
	<meta
		name="description"
		content="The QCU Consumers Cooperative is member-owned. Help, earn, and grow with services by the campus, for the campus."
	/>
</svelte:head>

<div
	id="top"
	class={theme}
	style="background: var(--bg); color: var(--text); font-family: 'Plus Jakarta Sans', system-ui, sans-serif; overflow-x: clip"
>
	<div
		style="position: sticky; top: 16px; z-index: 50; height: 0; display: flex; justify-content: center"
	>
		<nav
			class="navpill"
			aria-label="Main"
			style="box-sizing: border-box; height: 60px; max-width: calc(100% - 24px); display: flex; align-items: center; gap: 4px; padding: 0 10px; border-radius: 999px; background: var(--nav); backdrop-filter: blur(14px); -webkit-backdrop-filter: blur(14px); border: 1px solid var(--line); box-shadow: 0 12px 34px rgba(0,0,0,.25)"
		>
			<a href="#top" aria-label="Back to top" style="display: flex; flex-shrink: 0">
				<span class="brand-mark" role="img" aria-label="QCU Consumers Cooperative">QCU</span>
			</a>
			<div style="display: flex; overflow-x: auto; min-width: 0; scrollbar-width: none">
				<a class="navlink" href="#services">Services</a>
				<a class="navlink" href="#about">About Coop</a>
				<a class="navlink" href="#membership">Membership</a>
				<a class="navlink" href="#store">Store</a>
				<a class="navlink" href="#printing">Printing</a>
				<a class="navlink" href="#contact">Contact Us</a>
			</div>
			<button
				type="button"
				onclick={nextTheme}
				aria-label="Change theme"
				title={themeName}
				style="flex-shrink: 0; width: 40px; height: 40px; border-radius: 50%; border: 2px solid var(--accent); padding: 5px; background: transparent; cursor: pointer"
			>
				<span
					style="display: block; width: 100%; height: 100%; border-radius: 50%; background: {swatch}"
				></span>
			</button>
		</nav>
	</div>

	<header
		style="position: relative; min-height: 640px; display: flex; align-items: center; justify-content: center; overflow: hidden; color: #fff; text-align: center"
	>
		<div aria-hidden="true" style="position: absolute; inset: 0">
			<div class="slide hero-slide hero-one"></div>
			<div class="slide hero-slide hero-two" style="animation-delay: 9s"></div>
			<div class="slide hero-slide hero-three" style="animation-delay: 18s"></div>
		</div>
		<div style="position: absolute; inset: 0; background: var(--tint)"></div>
		<div
			style="position: relative; max-width: 900px; padding: 120px 20px 90px; display: flex; flex-direction: column; align-items: center; gap: 22px"
		>
			<h2
				aria-label="Quezon City University • Consumers Cooperative"
				style="margin: 0; font-size: 13px; font-weight: 800; letter-spacing: 3px; text-transform: uppercase; color: #ffd54a"
				use:headingTypewriter
			>
				<span class="type" aria-hidden="true">
					<span class="ghost">Quezon City University • Consumers Cooperative</span><span>
						{t.kicker}<span class="caret"></span>
					</span>
				</span>
			</h2>
			<h1
				aria-label="By the campus. For the campus."
				style="margin: 0; font-size: clamp(40px,7vw,78px); line-height: 1.08; font-weight: 800; letter-spacing: -2px"
				use:headingTypewriter
			>
				<span class="type" aria-hidden="true">
					<span class="ghost">By the campus. For the campus.</span><span>
						{heroText}<span class="caret"></span>
					</span>
				</span>
			</h1>
			<div style="display: flex; flex-wrap: wrap; justify-content: center; gap: 14px">
				<a class="btn" href="#membership">Become a member →</a>
				<a class="btn" href="#services">See our services ↓</a>
			</div>
		</div>
	</header>

	<div
		class="grid"
		style="box-sizing: border-box; max-width: 1120px; margin: -48px auto 0; padding: 0 20px; position: relative"
	>
		<div class="stat reveal" style="background: linear-gradient(135deg,#2457b8,#7a4fb5)">
			<div
				class="bg"
				style="background: radial-gradient(circle at 70% 20%,#f0a53a,#1f6f8f 70%)"
			></div>
			<div
				style="position: absolute; inset: 0; background: linear-gradient(180deg,transparent,var(--bg2) 100%); opacity: .5"
			></div>
			<div
				style="position: absolute; inset: 0; background: linear-gradient(180deg,transparent 20%,rgba(5,10,28,.85))"
			></div>
			<div
				role="img"
				aria-label="216"
				style="position: relative; font-size: 46px; font-weight: 800"
			>
				<span class="reel">
					<span class="strip" style="transform: translateY({d1}px)">
						{#each digits as n}<span>{n}</span>{/each}
					</span>
				</span><span class="reel">
					<span class="strip" style="transform: translateY({d2}px)">
						{#each digits as n}<span>{n}</span>{/each}
					</span>
				</span><span class="reel">
					<span class="strip" style="transform: translateY({d3}px)">
						{#each digits as n}<span>{n}</span>{/each}
					</span>
				</span>
			</div>
			<div style="position: relative; font-weight: 700">Member-owners</div>
		</div>
		<div class="stat reveal" style="background: linear-gradient(135deg,#12704f,#1d4f9c)">
			<div
				class="bg"
				style="animation-delay: 2s; background: radial-gradient(circle at 20% 20%,#49c58a,#20306b 70%)"
			></div>
			<div
				style="position: absolute; inset: 0; background: linear-gradient(180deg,transparent 20%,rgba(5,10,28,.85))"
			></div>
			<div
				role="img"
				aria-label="CDA"
				style="position: relative; font-size: 46px; font-weight: 800"
			>
				<span class="reel" style="width: 36px">
					<span class="strip" style="transform: translateY({l1}px)">
						<span>Q</span><span>M</span><span>X</span><span>B</span><span>C</span>
					</span>
				</span><span class="reel" style="width: 36px">
					<span class="strip" style="transform: translateY({l1}px)">
						<span>R</span><span>K</span><span>A</span><span>Z</span><span>D</span>
					</span>
				</span><span class="reel" style="width: 36px">
					<span class="strip" style="transform: translateY({l1}px)">
						<span>T</span><span>E</span><span>N</span><span>S</span><span>A</span>
					</span>
				</span>
			</div>
			<div style="position: relative; font-weight: 700">Registered &amp; compliant</div>
		</div>
		<div class="stat reveal" style="background: linear-gradient(135deg,#a8641a,#3b3f9a)">
			<div
				class="bg"
				style="animation-delay: 4s; background: radial-gradient(circle at 80% 30%,#e8c14a,#6b2f6f 70%)"
			></div>
			<div
				style="position: absolute; inset: 0; background: linear-gradient(180deg,transparent 20%,rgba(5,10,28,.85))"
			></div>
			<div role="img" aria-label="5" style="position: relative; font-size: 46px; font-weight: 800">
				<span class="reel">
					<span class="strip" style="transform: translateY({five}px)">
						{#each digits as n}<span>{n}</span>{/each}
					</span>
				</span>
			</div>
			<div style="position: relative; font-weight: 700">Campus services</div>
		</div>
		<div class="stat reveal" style="background: linear-gradient(135deg,#1f6f9f,#2a2f78)">
			<div
				class="bg"
				style="animation-delay: 6s; background: radial-gradient(circle at 30% 30%,#38c4d4,#262058 70%)"
			></div>
			<div
				style="position: absolute; inset: 0; background: linear-gradient(180deg,transparent 20%,rgba(5,10,28,.85))"
			></div>
			<div
				role="img"
				aria-label={currentDayIndex === null
					? 'Current day'
					: `${currentDayLabel}, ${isWeekday ? 'open weekdays' : 'closed'}`}
				style="position: relative; font-size: 46px; font-weight: 800"
			>
				<span class="reel">
					<span class="strip">
						<span>{currentDayLabel}</span>
					</span>
				</span>
			</div>
			<div style="position: relative; font-weight: 700">
				{#if currentDayIndex === null}
					Weekdays · 8:00 AM – 5:00 PM
				{:else if isWeekday}
					8:00 AM – 5:00 PM (Mon–Fri)
				{:else}
					Closed today · Mon–Fri, 8:00 AM – 5:00 PM
				{/if}
			</div>
		</div>
	</div>

	<section id="about" class="sec">
		<h3 class="eyebrow th" aria-label="Who we are?" use:headingTypewriter>
			<span class="type" aria-hidden="true">
				<span class="ghost">Who we are?</span><span>{t.who}<span class="caret"></span></span>
			</span>
		</h3>
		<h2 class="h2" use:headingTypewriter>A cooperative with a university's conscience</h2>
		<div class="grid" style="margin-top: 26px">
			<div class="card reveal">
				<h3 class="eyebrow th" aria-label="Mission" use:headingTypewriter>
					<span class="type" aria-hidden="true">
						<span class="ghost">Mission</span><span>{t.mission}<span class="caret"></span></span>
					</span>
				</h3>
				<p style="margin-top: 12px">
					To empower Quezon City University's community by providing accessible, reliable, and
					innovative cooperative services that enhance the well-being of our members.
				</p>
			</div>
			<div class="card reveal">
				<h3 class="eyebrow th" aria-label="Vision" use:headingTypewriter>
					<span class="type" aria-hidden="true">
						<span class="ghost">Vision</span><span>{t.vision}<span class="caret"></span></span>
					</span>
				</h3>
				<p style="margin-top: 12px">
					To be a leading university-based cooperative, recognized for integrity, service
					excellence, and positive impact on members' lives.
				</p>
			</div>
			<div class="card reveal">
				<h3 class="eyebrow th" aria-label="Our commitment" use:headingTypewriter>
					<span class="type" aria-hidden="true">
						<span class="ghost">Our commitment</span><span>
							{t.commit}<span class="caret"></span>
						</span>
					</span>
				</h3>
				<p style="margin-top: 12px">
					We operate with transparency and member-first policies, reinvesting surpluses into
					services, education, and community impact.
				</p>
			</div>
		</div>
		<h3 class="eyebrow" aria-label="Values" style="margin: 40px 0 16px" use:headingTypewriter>
			<span class="type" aria-hidden="true">
				<span class="ghost">Values</span><span>{t.values}<span class="caret"></span></span>
			</span>
		</h3>
		<div class="grid">
			<div class="card reveal">
				<h3 use:headingTypewriter>Community first</h3>
				<p>The welfare and development of members above all.</p>
			</div>
			<div class="card reveal">
				<h3 use:headingTypewriter>Bayanihan spirit</h3>
				<p>The Filipino tradition of helping one another.</p>
			</div>
			<div class="card reveal">
				<h3 use:headingTypewriter>Trust &amp; integrity</h3>
				<p>Transparent practices, responsible stewardship.</p>
			</div>
			<div class="card reveal">
				<h3 use:headingTypewriter>Excellence</h3>
				<p>High-quality service delivery, every day.</p>
			</div>
		</div>
	</section>

	<div style="background: var(--bg2)">
		<section id="services" class="sec">
			<h3 class="eyebrow th" aria-label="What we offer?" use:headingTypewriter>
				<span class="type" aria-hidden="true">
					<span class="ghost">What we offer?</span><span>{t.offer}<span class="caret"></span></span>
				</span>
			</h3>
			<h2 class="h2" use:headingTypewriter>Our Services</h2>
			<p class="lead">
				Services by the campus, for the campus — run by the same people behind your campus store,
				priced fairly, and accountable to the members who use them.
			</p>
			<div style="display: flex; flex-direction: column; gap: 14px; margin-top: 34px">
				<div class="slidein">
					<a class="row" href="#membership">
						<b class="eyebrow">01</b><span style="flex: 1">
							<b style="font-size: 19px">Dividends</b> <span class="eyebrow">Members</span><span
								style="display: block; color: var(--muted); margin-top: 6px"
							>
								Earn annual patronage refund and dividends as a member-owner — allocation,
								patronage, and surplus per policy.
							</span>
						</span><span class="go">→</span>
					</a>
				</div>
				<div class="slidein">
					<a class="row" href="#membership">
						<b class="eyebrow">02</b><span style="flex: 1">
							<b style="font-size: 19px">Regular Loans</b> <span class="eyebrow">Members</span><span
								style="display: block; color: var(--muted); margin-top: 6px"
							>
								Flexible personal loans with affordable rates, fast processing, and clear terms for
								eligible members.
							</span>
						</span><span class="go">→</span>
					</a>
				</div>
				<div class="slidein">
					<a class="row" href="#printing">
						<b class="eyebrow">03</b><span style="flex: 1">
							<b style="font-size: 19px">Printing Shop</b>
							<span class="eyebrow">Everyone</span><span
								style="display: block; color: var(--muted); margin-top: 6px"
							>
								Black &amp; color prints, photocopies, ID photos, and laminating — student-friendly
								pricing, on campus.
							</span>
						</span><span class="go">→</span>
					</a>
				</div>
				<div class="slidein">
					<a class="row" href="#store">
						<b class="eyebrow">04</b><span style="flex: 1">
							<b style="font-size: 19px">School Supplies</b>
							<span class="eyebrow">Everyone</span><span
								style="display: block; color: var(--muted); margin-top: 6px"
							>
								Affordable organizational supplies, paper products, and writing tools for QCU
								students.
							</span>
						</span><span class="go">→</span>
					</a>
				</div>
				<div class="slidein">
					<a class="row" href="#store">
						<b class="eyebrow">05</b><span style="flex: 1">
							<b style="font-size: 19px">Campus Store</b> <span class="eyebrow">Everyone</span><span
								style="display: block; color: var(--muted); margin-top: 6px"
							>
								Everyday essentials, beverages, and QCU merchandise at member-friendly prices.
							</span>
						</span><span class="go">→</span>
					</a>
				</div>
			</div>
		</section>
	</div>

	<section id="why" class="sec">
		<div class="why-layout">
			<div class="why-copy">
				<h3 class="eyebrow th" aria-label="Our purpose" use:headingTypewriter>
					<span class="type" aria-hidden="true">
						<span class="ghost">Our purpose</span><span>
							{t.purpose}<span class="caret"></span>
						</span>
					</span>
				</h3>
				<h2
					class="h2 th"
					aria-label="Why a coop in the university?"
					style="margin-top: 10px"
					use:headingTypewriter
				>
					<span class="type" aria-hidden="true">
						<span class="ghost">Why a coop in the university?</span><span>
							{t.why}<span class="caret"></span>
						</span>
					</span>
				</h2>
				<p class="lead">
					That's the whole idea behind Help. Earn. Grow. — any surplus returns to members or is
					reinvested into better services and education for the campus.
				</p>
				<div class="why-pillars grid" style="margin-top: 30px">
					<div class="card reveal">
						<div class="eyebrow">01</div>
						<h3 use:headingTypewriter>Community benefit</h3>
						<p>
							Convenient access on campus and shared savings from bulk purchasing — members pool
							resources to get fairly priced goods.
						</p>
					</div>
					<div class="card reveal">
						<div class="eyebrow">02</div>
						<h3 use:headingTypewriter>Financial inclusion</h3>
						<p>
							Member capital build-up and transparent, policy-based lending — a responsible
							alternative built around people, not profit.
						</p>
					</div>
					<div class="card reveal">
						<div class="eyebrow">03</div>
						<h3 use:headingTypewriter>Education mission</h3>
						<p>
							Cooperative values and real-world financial literacy, practiced inside a university —
							learning by owning.
						</p>
					</div>
				</div>
			</div>
			<div class="why-carousel">
				<div
					class="deck"
					role="group"
					aria-label="Swipe through campus and cooperative photos"
					onpointerdown={down}
					onpointerup={up}
				>
					<button
						type="button"
						class="photo photo-campus"
						aria-label="Campus life photo, swipe or tap for next"
						onclick={swipe}
						style="z-index: {c0.z}; transform: {c0.t}; opacity: {c0.o}"
					>
						Campus life<span style="font-size: 12px; font-weight: 500">QCU campus</span>
					</button>
					<button
						type="button"
						class="photo photo-members"
						aria-label="Member services photo, swipe or tap for next"
						onclick={swipe}
						style="z-index: {c1.z}; transform: {c1.t}; opacity: {c1.o}"
					>
						Member services<span style="font-size: 12px; font-weight: 500">Our cooperative</span>
					</button>
					<button
						type="button"
						class="photo photo-community"
						aria-label="Cooperative community photo, swipe or tap for next"
						onclick={swipe}
						style="z-index: {c2.z}; transform: {c2.t}; opacity: {c2.o}"
					>
						Cooperative community<span style="font-size: 12px; font-weight: 500">
							Members working together
						</span>
					</button>
				</div>
				<p class="why-caption">
					Serving the QCU community since our founding — on campus, in person.
				</p>
			</div>
		</div>
		<h3
			aria-label="How it works"
			style="margin: 48px 0 20px; font-size: 24px"
			use:headingTypewriter
		>
			<span class="type" aria-hidden="true">
				<span class="ghost">How it works:</span><span>{t.how1}<span class="caret"></span></span>
			</span>
		</h3>
		<div class="grid">
			<div class="slidein card">
				<div class="eyebrow">Step 1</div>
				<p style="margin-top: 10px">Members buy shares — capital build-up.</p>
			</div>
			<div class="slidein card">
				<div class="eyebrow">Step 2</div>
				<p style="margin-top: 10px">The coop operates services: store, loans, printing.</p>
			</div>
			<div class="slidein card">
				<div class="eyebrow">Step 3</div>
				<p style="margin-top: 10px">Surplus flows back — reserves, education, dividends.</p>
			</div>
		</div>
	</section>

	<div style="background: var(--bg2)">
		<section id="portal" class="sec split">
			<div class="reveal">
				<div
					class="window"
					role="img"
					aria-label="Preview of the member dashboard with sample figures"
				>
					<div class="bar">
						<span class="dot" style="background: #ff5f57"></span><span
							class="dot"
							style="background: #febc2e"
						></span><span class="dot" style="background: #28c840"></span><span
							style="margin-left: 8px"
						>
							QCU Cooperative · Member Dashboard
						</span>
					</div>
					<div style="display: flex">
						<div
							style="width: 108px; flex-shrink: 0; padding: 12px 8px; background: #ebeef5; font-size: 12px; display: flex; flex-direction: column; gap: 4px"
						>
							<span style="background: #1467e8; color: #fff; padding: 6px 10px; border-radius: 8px">
								Overview
							</span><span style="padding: 6px 10px">Shares</span><span style="padding: 6px 10px">
								Loans
							</span><span style="padding: 6px 10px">Dividends</span><span
								style="padding: 6px 10px"
							>
								Statements
							</span>
						</div>
						<div
							style="flex: 1; min-width: 0; padding: 12px; display: flex; flex-direction: column; gap: 10px"
						>
							<span
								style="align-self: flex-start; background: #d8f3e0; color: #12672f; font-size: 11px; font-weight: 700; padding: 4px 10px; border-radius: 999px"
							>
								● Member in good standing
							</span>
							<div
								style="display: grid; grid-template-columns: repeat(auto-fit,minmax(104px,1fr)); gap: 8px"
							>
								<div class="mini">Share Capital<b>₱48,250.00</b></div>
								<div class="mini">Dividends<b style="color: #137a37">₱3,412.75</b></div>
								<div class="mini">
									Loan Balance<b>₱12,000.00</b>
									<div style="height: 5px; background: #e6e9f1; margin-top: 6px">
										<div style="width: 25%; height: 100%; background: #1467e8"></div>
									</div>
									3 of 12 paid
								</div>
							</div>
							<div class="mini">
								<b style="font-size: 13px; margin: 0 0 6px">Recent activity</b>
								<div class="tx">
									<span>Jul 28 · Capital build-up</span><span style="color: #137a37">+₱500.00</span>
								</div>
								<div class="tx"><span>Jul 15 · Loan amortization</span><span>−₱1,100.00</span></div>
								<div class="tx"><span>Jul 08 · Store purchase</span><span>₱742.50</span></div>
								<div class="tx">
									<span>Jun 30 · Dividend credit</span><span style="color: #137a37">
										+₱3,412.75
									</span>
								</div>
							</div>
						</div>
					</div>
				</div>
				<div style="font-size: 12px; color: var(--muted); margin-top: 12px">
					Preview shown with sample figures.
				</div>
			</div>
			<div>
				<h3 class="eyebrow th" aria-label="Members Portal" use:headingTypewriter>
					<span class="type" aria-hidden="true">
						<span class="ghost">Members Portal</span><span>
							{t.portal}<span class="caret"></span>
						</span>
					</span>
				</h3>
				<h2
					class="h2 th"
					aria-label="Your coop, online"
					style="margin-top: 10px"
					use:headingTypewriter
				>
					<span class="type" aria-hidden="true">
						<span class="ghost">Your coop, online</span><span>
							{t.online}<span class="caret"></span>
						</span>
					</span>
				</h2>
				<p class="lead fu" style="margin-bottom: 26px">
					Members can check share capital, dividends, and loan status anytime — records, receipts,
					and reports without a trip to the office.
				</p>
				<a class="btn" href="https://qcu-membership.vercel.app/" target="_blank" rel="noopener">
					Members Login →
				</a>
			</div>
		</section>
	</div>

	<div
		id="membership"
		class="anchor"
		style="position: relative; overflow: hidden; color: #fff; text-align: center"
	>
		<div
			class="para membership-backdrop"
			aria-hidden="true"
			style="position: absolute; inset: -60px 0"
		></div>
		<div style="position: absolute; inset: 0; background: var(--tint)"></div>
		<div
			style="position: relative; padding: 100px 20px; display: flex; flex-direction: column; align-items: center; gap: 14px"
		>
			<h3
				aria-label="Membership"
				style="margin: 0; font-size: 13px; font-weight: 800; letter-spacing: 3px; text-transform: uppercase; color: #ffd54a"
				use:headingTypewriter
			>
				<span class="type" aria-hidden="true">
					<span class="ghost">Membership</span><span>{t.member}<span class="caret"></span></span>
				</span>
			</h3>
			<p style="margin: 0; max-width: 660px; font-size: 18px; line-height: 1.7; color: #e3e9fb">
				The QCU Consumers Cooperative is member-owned. Every peso our services earn goes back into
				the cooperative and the QCU community — not to an outside business.
			</p>
			<h2
				class="h2 th"
				aria-label="Five steps to joining"
				style="margin: 0; opacity: {joinFade}; transition: opacity .6s"
				use:headingTypewriter
			>
				<span class="type" aria-hidden="true">
					<span class="ghost">Five steps to joining</span><span>
						{joinText}<span class="caret"></span>
					</span>
				</span>
			</h2>
			<p style="margin: 0; max-width: 620px; line-height: 1.7; color: #e3e9fb">
				Primarily open to QCU employees per coop policy; student programs may be offered — inquire
				at the office.
			</p>
			<a class="btn" href="#receipt">See the print membership receipt ↓</a>
		</div>
	</div>

	<section class="sec">
		<div class="grid">
			<div class="card reveal">
				<div class="eyebrow">01</div>
				<h3 use:headingTypewriter>Check eligibility</h3>
				<p>
					Generally open to QCU employees (e.g., with at least 2 years of service). Confirm criteria
					with the coop office.
				</p>
			</div>
			<div class="card reveal">
				<div class="eyebrow">02</div>
				<h3 use:headingTypewriter>Prepare requirements</h3>
				<p>
					Valid ID, QCU Employee ID, recent photo, and initial share capital as prescribed by
					policy.
				</p>
			</div>
			<div class="card reveal">
				<div class="eyebrow">03</div>
				<h3 use:headingTypewriter>Submit your application</h3>
				<p>File your documents at the coop office, or reach us by email or phone.</p>
			</div>
			<div class="card reveal">
				<div class="eyebrow">04</div>
				<h3 use:headingTypewriter>Orientation &amp; approval</h3>
				<p>
					Attend orientation if scheduled; applications are reviewed per policy before activation.
				</p>
			</div>
			<div class="card reveal">
				<div class="eyebrow">05</div>
				<h3 use:headingTypewriter>Save &amp; access services</h3>
				<p>Begin your capital build-up and access member services. Welcome aboard.</p>
			</div>
		</div>
		<div class="split" style="margin-top: 64px; align-items: start">
			<div>
				<h3
					aria-label="Requirements"
					style="margin: 0 0 18px; font-size: 26px"
					use:headingTypewriter
				>
					<span class="type" aria-hidden="true">
						<span class="ghost">Requirements:</span><span>{t.reqs}<span class="caret"></span></span>
					</span>
				</h3>
				<div style="display: flex; flex-direction: column; gap: 10px">
					<div class="slidein card" style="padding: 16px 20px">
						<span style="color: var(--accent)">✓</span> Valid government ID
					</div>
					<div class="slidein card" style="padding: 16px 20px">
						<span style="color: var(--accent)">✓</span> QCU Employee ID
					</div>
					<div class="slidein card" style="padding: 16px 20px">
						<span style="color: var(--accent)">✓</span> Recent 1×1 photo
					</div>
					<div class="slidein card" style="padding: 16px 20px">
						<span style="color: var(--accent)">✓</span> Initial share capital &amp; membership fee
					</div>
				</div>
				<p style="font-size: 13px; color: var(--muted)">
					Amounts are set by policy and may change — confirm at the coop office.
				</p>
			</div>
			<div>
				<h3
					aria-label="Common questions"
					style="margin: 0 0 18px; font-size: 26px"
					use:headingTypewriter
				>
					<span class="type" aria-hidden="true">
						<span class="ghost">Common questions</span><span>
							{t.faq}<span class="caret"></span>
						</span>
					</span>
				</h3>
				<div style="display: flex; flex-direction: column; gap: 10px">
					<details>
						<summary>Who can join?</summary>
						<p>
							Primarily QCU employees, subject to coop policies. Please check with the office for
							any updates.
						</p>
					</details>
					<details>
						<summary>How much is the initial share capital?</summary>
						<p>
							Set by policy and may change from time to time. Contact the coop office for current
							amounts and payment options.
						</p>
					</details>
					<details>
						<summary>Can students join?</summary>
						<p>
							Student participation may be offered via specific programs. Please inquire with the
							coop for availability.
						</p>
					</details>
					<details>
						<summary>Where do I submit requirements?</summary>
						<p>
							File your documents at the coop office during business hours (Mon–Fri, 8:00 AM – 5:00
							PM), or reach us by email or phone.
						</p>
					</details>
				</div>
			</div>
		</div>
		<div
			class="promo reveal"
			style="margin-top: 64px; text-align: center; background: linear-gradient(120deg,#16306e,#2457b8,#12704f)"
		>
			<div class="orb" style="width: 240px; height: 240px; left: -60px; top: -90px"></div>
			<div class="orb" style="width: 200px; height: 200px; right: -40px; bottom: -90px"></div>
			<h3
				class="fu"
				style="position: relative; margin: 0 0 8px; font-size: 30px"
				use:headingTypewriter
			>
				Ready to be a member-owner?
			</h3>
			<p style="position: relative; margin: 0 0 22px; line-height: 1.6">
				Help your community, earn from what you already spend, and grow your savings — help · earn ·
				grow.
			</p>
			<a class="btn" href="#contact" style="position: relative">Visit the Coop Office &gt;</a>
		</div>
	</section>

	<div style="background: var(--bg2)">
		<section id="store" class="sec split">
			<div>
				<h3 class="eyebrow th" aria-label="Store Portal" use:headingTypewriter>
					<span class="type" aria-hidden="true">
						<span class="ghost">Store Portal</span><span>{t.store}<span class="caret"></span></span>
					</span>
				</h3>
				<h2 class="h2" use:headingTypewriter>Your coop, accessible.</h2>
				<p class="lead fu" style="margin-bottom: 26px">
					Customers can check products, cart, and purchase history anytime — accessible in a single
					click before going to the store.
				</p>
				<a class="btn" href="https://qcu-coop-zeta.vercel.app/" target="_blank" rel="noopener">
					Members Login →
				</a>
			</div>
			<div class="reveal">
				<div
					class="window"
					role="img"
					aria-label="Preview of the QCU Coop Store"
					style="background: #e6f1fd"
				>
					<div class="bar">
						<span class="dot" style="background: #ff5f57"></span><span
							class="dot"
							style="background: #febc2e"
						></span><span class="dot" style="background: #28c840"></span><span
							style="margin-left: 8px"
						>
							QCU COOP STORE
						</span>
					</div>
					<div style="padding: 20px">
						<span
							style="font-size: 10px; font-weight: 800; letter-spacing: 1px; color: #7a3d00; background: #fff1c2; padding: 5px 10px; border-radius: 999px"
						>
							MEMBER SHOP • A.Y. 2026–2027
						</span>
						<div style="font-size: 24px; font-weight: 800; line-height: 1.2; margin: 14px 0">
							Everything for the semester, plus <span style="color: #1553d1">your share back.</span>
						</div>
						<div style="display: grid; grid-template-columns: repeat(3,minmax(0,1fr)); gap: 10px">
							<div class="mini">
								<div style="height: 70px; border-radius: 8px; background: #d7e5f7"></div>
								<b style="font-size: 12px">Ballpen - Blue</b><b style="font-size: 14px">₱10.00</b>
							</div>
							<div class="mini" style="box-shadow: 0 10px 24px rgba(20,70,160,.2)">
								<div style="height: 70px; border-radius: 8px; background: #d7e5f7"></div>
								<b style="font-size: 12px">Band-aid w/ cotton balls</b><b style="font-size: 14px">
									₱3.00
								</b>
							</div>
							<div class="mini">
								<div style="height: 70px; border-radius: 8px; background: #d7e5f7"></div>
								<b style="font-size: 12px">Binder clips</b><b style="font-size: 14px">View →</b>
							</div>
						</div>
					</div>
				</div>
			</div>
		</section>
	</div>

	<section id="printing" class="sec">
		<div id="receipt" class="split anchor">
			<div>
				<h3
					class="eyebrow th"
					aria-label="Print membership · Official receipt"
					use:headingTypewriter
				>
					<span class="type" aria-hidden="true">
						<span class="ghost">Print membership · Official receipt</span><span>
							{t.receipt}<span class="caret"></span>
						</span>
					</span>
				</h3>
				<h2 class="h2" style="font-size: clamp(42px,6vw,68px)" use:headingTypewriter>
					One receipt.<br />Whole degree.
				</h2>
				<p class="lead fu">
					Print membership is <b style="color: var(--accent); font-size: 24px">₱50</b> <s>₱100</s>,
					paid once — it doesn't lapse, doesn't renew, and only expires the day you graduate.
				</p>
			</div>
			<div
				class="reveal"
				style="width: 100%; max-width: 340px; margin: 0 auto; transform: rotate(2deg); font-family: 'JetBrains Mono', monospace; color: #1c1c1c; filter: drop-shadow(0 24px 30px rgba(0,0,0,.3))"
			>
				<div class="zig" style="transform: scaleY(-1)"></div>
				<div style="background: #fffdf6; padding: 18px 26px; font-size: 13px; line-height: 2">
					<div style="text-align: center; font-weight: 700; line-height: 1.5">
						QCU CONSUMERS COOPERATIVE<br />PRINT MEMBERSHIP<br />— OFFICIAL RECEIPT —
					</div>
					<div style="border-top: 2px dashed #999; margin: 12px 0"></div>
					<div style="display: flex; justify-content: space-between">
						<span>REGULAR</span><s>₱100.00</s>
					</div>
					<div style="display: flex; justify-content: space-between">
						<span>PAID</span><span>ONCE</span>
					</div>
					<div style="display: flex; justify-content: space-between">
						<span>RENEWAL</span><span>NONE</span>
					</div>
					<div style="display: flex; justify-content: space-between">
						<span>VALID UNTIL</span><span>GRADUATION</span>
					</div>
					<div style="border-top: 2px dashed #999; margin: 12px 0"></div>
					<div
						style="display: flex; justify-content: space-between; font-size: 22px; font-weight: 700"
					>
						<span>TOTAL</span><span>₱50.00</span>
					</div>
					<div style="margin: 14px 0; text-align: center">
						<span
							style="display: inline-block; border: 3px double #b3261e; color: #b3261e; padding: 2px 14px; font-weight: 700; transform: rotate(-9deg)"
						>
							ONE-TIME · SAMPLE
						</span>
					</div>
					<div class="bars" aria-hidden="true"></div>
				</div>
				<div class="zig"></div>
			</div>
		</div>

		<div class="printing-intro">
			<h2 class="h2" use:headingTypewriter>Printing that gives back to campus</h2>
			<p class="lead">
				The QCU Consumers Cooperative is member-owned — the same people who run your campus store.
				Every peso the printing service earns goes back into the cooperative and the QCU community,
				not to an outside business.
			</p>
			<p class="lead fu" style="margin-top: 16px">
				That's the whole idea behind Help. Earn. Grow. — services by the campus, for the campus.
			</p>
		</div>

		<div style="margin-top: 96px">
			<h3 class="eyebrow th" aria-label="Printing Services" use:headingTypewriter>
				<span class="type" aria-hidden="true">
					<span class="ghost">Printing Services</span><span>
						{t.services}<span class="caret"></span>
					</span>
				</span>
			</h3>
			<h2 class="h2" use:headingTypewriter>The basics, done right.</h2>
			<p class="lead fu">
				We're starting with the everyday things students actually need. Prices will be posted at the
				counter when we open.
			</p>
			<div class="grid" style="margin-top: 28px">
				<div class="card reveal">
					<img
						class="service-image"
						src="/coop/black&white_printing-img.jpeg"
						alt="Black-and-white pages coming out of a printer"
					/>
					<h3 use:headingTypewriter>Black &amp; white printing</h3>
					<p>
						Essays, reports, readings, and reviewers — short and long bond, single or double-sided.
					</p>
				</div>
				<div class="card reveal">
					<img
						class="service-image"
						src="/coop/colored_printing-img.jpeg"
						alt="Color prints being checked for accurate colors"
					/>
					<h3 use:headingTypewriter>Color printing</h3>
					<p>
						Figures, posters for class, presentation handouts, and anything that needs to look its
						best.
					</p>
				</div>
				<div class="card reveal">
					<img
						class="service-image"
						src="/coop/photocopy-img.jpeg"
						alt="A photocopier copying a document"
					/>
					<h3 use:headingTypewriter>Photocopying</h3>
					<p>Copies of notes, forms, and requirements — one page or a whole set for your block.</p>
				</div>
				<div class="card reveal">
					<div style="font-size: 34px; font-weight: 800; color: var(--accent)">₱</div>
					<h3 use:headingTypewriter>Student-friendly pricing</h3>
					<p>Fair per-page rates set by your coop — posted at the counter when we open.</p>
				</div>
			</div>
		</div>

		<div style="margin-top: 80px">
			<h3 class="eyebrow th" aria-label="How it works?" use:headingTypewriter>
				<span class="type" aria-hidden="true">
					<span class="ghost">How it works?</span><span>{t.how2}<span class="caret"></span></span>
				</span>
			</h3>
			<h2 class="h2" use:headingTypewriter>Three steps between you and your printout</h2>
			<div class="grid" style="margin-top: 24px">
				<div class="card slidein">
					<span
						style="display: inline-flex; width: 44px; height: 44px; border-radius: 50%; align-items: center; justify-content: center; background: var(--btn); color: var(--btntext); font-weight: 800; animation: pulse 2.4s infinite"
					>
						1
					</span>
					<h3 use:headingTypewriter>Bring your file</h3>
					<p>USB or phone — PDF keeps your layout intact.</p>
				</div>
				<div class="card slidein">
					<span
						style="display: inline-flex; width: 44px; height: 44px; border-radius: 50%; align-items: center; justify-content: center; background: var(--btn); color: var(--btntext); font-weight: 800"
					>
						2
					</span>
					<h3 use:headingTypewriter>Pay at the counter</h3>
					<p>Posted price by paper size, color, and copies.</p>
				</div>
				<div class="card slidein">
					<span
						style="display: inline-flex; width: 44px; height: 44px; border-radius: 50%; align-items: center; justify-content: center; background: var(--btn); color: var(--btntext); font-weight: 800"
					>
						3
					</span>
					<h3 use:headingTypewriter>Pick up and go</h3>
					<p>Most jobs print while you wait — batches by pickup.</p>
				</div>
			</div>
		</div>

		<div style="margin-top: 80px">
			<h3 class="eyebrow th" aria-label="Printing Promos" use:headingTypewriter>
				<span class="type" aria-hidden="true">
					<span class="ghost">Printing Promos</span><span>
						{t.promos}<span class="caret"></span>
					</span>
				</span>
			</h3>
			<h2 class="h2" use:headingTypewriter>Perks for the QCU community.</h2>
			<div class="grid" style="margin-top: 24px">
				<div class="promo reveal" style="background: linear-gradient(135deg,#1346b0,#0a79d8)">
					<div class="orb" style="width: 200px; height: 200px; right: -50px; top: -60px"></div>
					<div class="orb" style="width: 150px; height: 150px; right: 40px; top: 30px"></div>
					<div style="position: relative">
						<b>Coop members</b>
						<div style="font-size: 64px; font-weight: 800">25% off</div>
						<p style="margin: 0">
							Members save on every print job — one more reason to join your coop.
						</p>
					</div>
				</div>
				<div class="promo reveal" style="background: linear-gradient(135deg,#7a3a12,#a8641a)">
					<div class="orb" style="width: 180px; height: 180px; right: -40px; bottom: -60px"></div>
					<div class="orb" style="width: 130px; height: 130px; right: 60px; bottom: -20px"></div>
					<div style="position: relative">
						<b>Faculty &amp; professors</b>
						<div style="font-size: 64px; font-weight: 800">18% off</div>
					</div>
				</div>
			</div>
		</div>

		<div id="wallet" class="split anchor" style="margin-top: 96px; align-items: start">
			<div>
				<h3 class="eyebrow th" aria-label="Wallet · Print &amp; Earn Rewards" use:headingTypewriter>
					<span class="type" aria-hidden="true">
						<span class="ghost">Wallet · Print &amp; Earn Rewards</span><span>
							{t.wallet}<span class="caret"></span>
						</span>
					</span>
				</h3>
				<h2 class="h2" use:headingTypewriter>Print now,<br />earn as you go.</h2>
				<div style="display: flex; align-items: center; gap: 14px; margin: 20px 0">
					<span
						style="font-size: 12px; font-weight: 800; padding: 6px 12px; border-radius: 999px; border: 1px solid var(--accent); color: var(--accent)"
					>
						New
					</span>
					<span class="loyalty-mark" role="img" aria-label="Limbag Gantimpala">LG</span>
					<div>
						<h3
							class="eyebrow th"
							aria-label="Introducing"
							style="margin: 0; color: var(--muted)"
							use:headingTypewriter
						>
							<span class="type" aria-hidden="true">
								<span class="ghost">Introducing</span><span>
									{t.intro}<span class="caret"></span>
								</span>
							</span>
						</h3>
						<div style="font-size: 28px; font-weight: 800">
							Limbag <span style="color: var(--accent)">Gantimpala</span>
						</div>
					</div>
				</div>
				<p class="lead fu" style="margin-bottom: 12px">
					<b>Your loyalty pass, run by your cooperative.</b>
				</p>
				<p class="lead">
					Ask the printing counter to enroll you and you get a loyalty pass — a private link you
					open on your phone. Every print job adds points, and one point is ₱1 off what you pay next
					time.
				</p>
				<div style="display: flex; flex-wrap: wrap; gap: 28px; margin: 22px 0">
					<div><b style="color: var(--accent); font-size: 20px">₱16.88</b> spent earns 1 point</div>
					<div>
						<b style="color: var(--accent); font-size: 20px">1 point</b> is ₱1 off at the counter
					</div>
				</div>
				<a class="btn" href="#findus">How to join →</a>
				<div style="margin-top: 16px; font-size: 14px">
					Already a member? <a href="#contact" style="font-weight: 800; text-decoration: underline">
						Get your wallet link →
					</a>
				</div>
			</div>
			<div
				style="width: 100%; max-width: 460px; margin: 0 auto; display: flex; flex-direction: column; gap: 16px"
			>
				<div class="stage">
					<div class="pass">
						<span class="face" style="display: block">
							<span style="display: flex; align-items: center; gap: 12px">
								<span class="loyalty-mark" aria-hidden="true">LG</span><span style="flex: 1">
									<b style="display: block">Limbag Gantimpala</b><span style="font-size: 12px">
										Print &amp; Earn Rewards · Loyalty pass
									</span>
								</span><b
									style="font-size: 12px; background: rgba(255,255,255,.18); padding: 6px 12px; border-radius: 8px"
								>
									CLIENT
								</b>
							</span>
							<span style="display: block; margin: 18px 0">
								<b style="font-size: 52px">268.50</b> points
								<span style="font-size: 12px">· 1 pt = ₱1 at the counter</span>
							</span>
							<span style="display: flex; justify-content: space-between; font-size: 12px">
								<span>
									CLIENT NAME<b style="display: block; font-size: 16px">Sample Client</b>
								</span><span style="text-align: right">
									STUDENT NO.<b style="display: block; font-size: 16px">0000-0000</b>
								</span>
							</span>
							<span style="display: block; margin-top: 14px; font-size: 12px">
								Earning today: every ₱16.88 spent = 1 point
							</span>
						</span>
						<span style="display: block; padding: 16px 50px; text-align: center">
							<span class="bars" style="display: block"></span><span
								style="display: block; font-size: 13px; letter-spacing: 6px; margin-top: 6px"
							>
								00000000
							</span><span style="font-size: 12px; color: #555">
								Show this at the counter to earn points.
							</span>
						</span>
					</div>
				</div>
				<div class="card" style="padding: 8px 20px">
					<div class="eyebrow" style="padding: 12px 0">History</div>
					<div
						style="display: flex; justify-content: space-between; padding: 12px 0; border-top: 1px solid var(--line)"
					>
						<span>
							Printing · ₱40.00<span style="display: block; font-size: 12px; color: var(--muted)">
								Today, 10:24 AM
							</span>
						</span><b>+2.37</b>
					</div>
					<div
						style="display: flex; justify-content: space-between; padding: 12px 0; border-top: 1px solid var(--line)"
					>
						<span>
							₱25.00 off your print<span
								style="display: block; font-size: 12px; color: var(--muted)"
							>
								Friday, 2:05 PM
							</span>
						</span><b>−25.00</b>
					</div>
				</div>
				<div style="font-size: 12px; color: var(--muted); text-align: center">
					Preview shown with sample figures — your own pass shows your real balance and your own
					barcode.
				</div>
			</div>
		</div>
	</section>

	<div
		id="findus"
		class="anchor"
		style="position: relative; overflow: hidden; color: #fff; background: #0a1520"
	>
		<svg
			viewBox="0 0 1200 440"
			preserveAspectRatio="xMidYMid slice"
			aria-hidden="true"
			style="position: absolute; inset: 0; width: 100%; height: 100%; opacity: .55"
		>
			<g fill="#17301f">
				<path d="M0 0H360L300 150L0 190Z"></path><path d="M760 300L1200 240V440H800Z"></path>
			</g>
			<g fill="#233246">
				<rect x="420" y="60" width="120" height="70"></rect><rect
					x="580"
					y="90"
					width="80"
					height="130"
				></rect><rect x="700" y="50" width="150" height="80"></rect><rect
					x="900"
					y="100"
					width="110"
					height="90"
				></rect><rect x="380" y="270" width="160" height="90"></rect><rect
					x="590"
					y="300"
					width="100"
					height="80"
				></rect>
			</g>
			<g stroke="#52667f" fill="none">
				<path d="M0 240L1200 170" stroke-width="18"></path><path
					d="M350 0L300 440"
					stroke-width="10"
				></path><path d="M720 0L780 440" stroke-width="8"></path>
			</g>
		</svg>
		<div
			style="position: absolute; inset: 0; background: linear-gradient(90deg,rgba(5,12,26,.92),rgba(5,12,26,.55))"
		></div>
		<div class="sec split" style="position: relative">
			<div>
				<h3
					aria-label="About"
					style="margin: 0; font-size: 13px; font-weight: 800; letter-spacing: 3px; text-transform: uppercase; color: #ffd54a"
					use:headingTypewriter
				>
					<span class="type" aria-hidden="true">
						<span class="ghost">About</span><span>{t.about}<span class="caret"></span></span>
					</span>
				</h3>
				<h2 class="h2 th" aria-label="Find us!" style="margin-top: 10px" use:headingTypewriter>
					<span class="type" aria-hidden="true">
						<span class="ghost">Find us!</span><span>{t.find}<span class="caret"></span></span>
					</span>
				</h2>
				<p style="line-height: 1.7; color: #dbe4f8">
					Lower ground, Bautista Building. Same home as your coop store — Quezon City University,
					San Bartolome, Novaliches. Open on school days; exact hours will be posted here and at the
					door when we launch.
				</p>
				<div style="display: flex; align-items: center; gap: 16px; margin: 22px 0">
					<span
						style="display: inline-flex; width: 56px; height: 56px; border-radius: 50%; align-items: center; justify-content: center; background: #ffd54a; color: #17203f; font-weight: 800; animation: pulse 2.4s infinite"
					>
						LG
					</span><b>Bautista Building, QCU Main Campus</b>
				</div>
				<a
					class="btn"
					href="https://www.google.com/maps/search/?api=1&amp;query=Quezon+City+University+Main+Campus"
					target="_blank"
					rel="noopener"
				>
					Open in Google Maps ↗
				</a>
			</div>
			<div
				style="width: 100%; max-width: 340px; margin: 0 auto; border-radius: 20px; overflow: hidden; background: #fff; color: #16213f; text-align: center; box-shadow: 0 30px 70px rgba(0,0,0,.5); animation: float 7s ease-in-out infinite"
			>
				<div class="bar">
					<span class="dot" style="background: #ff5f57"></span><span
						class="dot"
						style="background: #febc2e"
					></span><span class="dot" style="background: #28c840"></span><span
						style="margin-left: 8px"
					>
						Live printing status
					</span>
				</div>
				<div style="padding: 24px">
					<h3 style="margin: 0 0 16px; font-size: 22px" use:headingTypewriter>
						Are we open right now?
					</h3>
					<img
						src="/coop/qr-printing-status.png"
						alt="QR code to check the printing counter's live status"
						width="180"
						height="180"
						style="display: block; width: 180px; height: 180px; margin: 0 auto; border-radius: 14px"
					/>
					<p style="margin: 16px 0 0; font-size: 14px; line-height: 1.6; color: #55607c">
						Scan to check the live open / closed status of the printing counter before you walk
						over.
					</p>
				</div>
			</div>
		</div>
	</div>

	<footer id="contact" class="anchor" style="background: #061d3f; color: #c8d6f0">
		<div
			class="grid"
			style="box-sizing: border-box; max-width: 1120px; margin: 0 auto; padding: 64px 20px 36px; font-size: 14px; line-height: 1.7"
		>
			<div>
				<span class="footer-brand-mark" role="img" aria-label="QCU Consumers Cooperative">QCU</span>
				<div style="font-size: 18px; font-weight: 800; color: #fff; margin: 10px 0">
					QCU Consumers Cooperative
				</div>
				<div>
					Together we <b style="color: #fff">help</b>, <b style="color: #ffd54a">earn</b>, and
					<b style="color: #3fdc8c">grow</b> — for a stronger QCU community.
				</div>
				<a
					href="https://www.facebook.com/QCUCoopstore"
					target="_blank"
					rel="noopener"
					style="color: #fff; font-weight: 700"
				>
					facebook.com/QCUCoopstore
				</a>
			</div>
			<div style="display: flex; flex-direction: column; gap: 8px; align-items: flex-start">
				<h3
					aria-label="Explore"
					style="margin: 0; font-size: 14px; letter-spacing: 3px; text-transform: uppercase"
					use:headingTypewriter
				>
					<span class="type" aria-hidden="true">
						<span class="ghost">Explore</span><span>{t.explore}<span class="caret"></span></span>
					</span>
				</h3>
				<a href="#services" style="color: #c8d6f0">Our Services</a><a
					href="#why"
					style="color: #c8d6f0"
				>
					Why a Coop
				</a><a href="#about" style="color: #c8d6f0">About</a><a
					href="#membership"
					style="color: #c8d6f0"
				>
					Membership
				</a>
			</div>
			<div style="display: flex; flex-direction: column; gap: 8px; align-items: flex-start">
				<h3
					aria-label="Portals"
					style="margin: 0; font-size: 14px; letter-spacing: 3px; text-transform: uppercase"
					use:headingTypewriter
				>
					<span class="type" aria-hidden="true">
						<span class="ghost">Portals</span><span>{t.portals}<span class="caret"></span></span>
					</span>
				</h3>
				<a
					href="https://qcu-membership.vercel.app/"
					target="_blank"
					rel="noopener"
					style="color: #c8d6f0"
				>
					Members Login
				</a><span>Employees Login</span><span>Identra — Staff Portal ↗</span><span
					style="color: #8fa8d4"
				>
					Coop Store — coming soon
				</span><a
					href="https://registration.gradpic.qcucooperative.com/000"
					target="_blank"
					rel="noopener"
					style="color: #c8d6f0"
				>
					Grad Pic Registration ↗
				</a>
			</div>
			<div>
				<h3
					aria-label="Contact"
					style="margin: 0; font-size: 14px; letter-spacing: 3px; text-transform: uppercase"
					use:headingTypewriter
				>
					<span class="type" aria-hidden="true">
						<span class="ghost">Contact</span><span>{t.contact}<span class="caret"></span></span>
					</span>
				</h3>
				<p>
					LG, Bautista Building, Quezon City University<br />673 Quirino Highway, San Bartolome,
					Novaliches, Quezon City
				</p>
				<a href="tel:+639983969387" style="display: block; color: #fff">+63 998 396 9387</a><a
					href="mailto:qcumultipurposecooperative@yahoo.com"
					style="display: block; color: #fff; overflow-wrap: anywhere"
				>
					qcumultipurposecooperative@yahoo.com
				</a>
			</div>
		</div>
		<div
			style="box-sizing: border-box; max-width: 1120px; margin: 0 auto; padding: 20px; border-top: 1px solid rgba(255,255,255,.14); display: flex; flex-wrap: wrap; justify-content: space-between; align-items: center; gap: 14px; font-size: 13px"
		>
			<span>© 2026 Quezon City University Consumers Cooperative. All rights reserved.</span><a
				class="btn"
				href="#top"
				aria-label="Back to top"
				style="width: 48px; padding: 0"
			>
				↑
			</a>
		</div>
	</footer>
</div>

<style>
	:global(body) {
		margin: 0;
	}
	.gold {
		--scrim: rgba(23, 31, 63, 0.95);
		--bg: #0a1126;
		--bg2: #101a36;
		--card: #17213f;
		--text: #fff;
		--muted: #bcc6e2;
		--line: rgba(255, 255, 255, 0.14);
		--accent: #ffd04a;
		--btn: linear-gradient(110deg, #f2b824, #ffe27d, #f2a824);
		--btntext: #17203f;
		--nav: rgba(13, 21, 45, 0.72);
		--tint: linear-gradient(180deg, rgba(8, 14, 34, 0.78), rgba(14, 24, 56, 0.84));
	}
	.royal {
		--scrim: rgba(8, 26, 84, 0.95);
		--bg: #06102e;
		--bg2: #0b1a47;
		--card: #11245c;
		--text: #fff;
		--muted: #bacbf2;
		--line: rgba(255, 255, 255, 0.15);
		--accent: #5fd2ff;
		--btn: linear-gradient(110deg, #4cc9ff, #a5e8ff, #38b6ff);
		--btntext: #06163a;
		--nav: rgba(8, 20, 58, 0.72);
		--tint: linear-gradient(180deg, rgba(5, 11, 36, 0.78), rgba(14, 40, 110, 0.82));
	}
	.sky {
		--scrim: rgba(18, 66, 150, 0.95);
		--bg: #f3f8ff;
		--bg2: #e3efff;
		--card: #fff;
		--text: #10214a;
		--muted: #43557f;
		--line: rgba(16, 33, 74, 0.14);
		--accent: #1253bd;
		--btn: linear-gradient(110deg, #1253bd, #2270d6, #0f4aa8);
		--btntext: #fff;
		--nav: rgba(255, 255, 255, 0.74);
		--tint: linear-gradient(180deg, rgba(8, 20, 50, 0.72), rgba(14, 36, 86, 0.8));
	}
	a {
		color: var(--accent);
		text-decoration: none;
	}
	a:hover {
		color: var(--text);
	}
	.sec {
		box-sizing: border-box;
		max-width: 1120px;
		margin: 0 auto;
		padding: 88px 20px;
		scroll-margin-top: 90px;
	}
	.anchor {
		scroll-margin-top: 90px;
	}
	.eyebrow {
		font-size: 12px;
		font-weight: 800;
		letter-spacing: 3px;
		text-transform: uppercase;
		color: var(--accent);
	}
	.h2 {
		margin: 10px 0 14px;
		font-size: clamp(30px, 4.4vw, 46px);
		line-height: 1.12;
		font-weight: 800;
		letter-spacing: -1px;
	}
	.lead {
		margin: 0;
		font-size: 17px;
		line-height: 1.7;
		color: var(--muted);
		max-width: 640px;
	}
	.grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(min(100%, 240px), 1fr));
		gap: 18px;
	}
	.split {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(min(100%, 440px), 1fr));
		gap: 48px;
		align-items: center;
	}
	.card {
		background: var(--card);
		border: 1px solid var(--line);
		border-radius: 22px;
		padding: 24px;
		transition:
			transform 0.35s cubic-bezier(0.2, 1.4, 0.4, 1),
			box-shadow 0.35s,
			border-color 0.35s;
	}
	.card:hover {
		transform: translateY(-6px);
		box-shadow: 0 22px 44px rgba(0, 0, 0, 0.25);
		border-color: var(--accent);
	}
	.card h3 {
		margin: 10px 0 8px;
		font-size: 19px;
	}
	.card p {
		margin: 0;
		color: var(--muted);
		font-size: 15px;
		line-height: 1.65;
	}
	.btn {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		gap: 8px;
		min-height: 48px;
		padding: 0 26px;
		border: 0;
		border-radius: 999px;
		background: var(--btn);
		background-size: 220% 100%;
		color: var(--btntext);
		font:
			800 15px 'Plus Jakarta Sans',
			system-ui,
			sans-serif;
		cursor: pointer;
		animation: shine 5s ease infinite;
		transition:
			transform 0.25s,
			box-shadow 0.25s;
	}
	.btn:hover {
		color: var(--btntext);
		transform: translateY(-3px) scale(1.03);
		box-shadow: 0 14px 30px rgba(0, 0, 0, 0.3);
	}
	.btn:active {
		transform: scale(0.97);
	}
	.navlink {
		display: inline-flex;
		align-items: center;
		min-height: 40px;
		padding: 0 12px;
		border-radius: 999px;
		color: var(--text);
		font-size: 14px;
		font-weight: 700;
		white-space: nowrap;
	}
	.navlink {
		transition: transform 0.25s;
	}
	.navlink:hover {
		background: var(--btn);
		color: var(--btntext);
		transform: translateY(-2px);
	}
	.navpill {
		animation: drop 0.8s cubic-bezier(0.2, 1.4, 0.4, 1) both;
	}
	@keyframes drop {
		from {
			opacity: 0;
			transform: translateY(-80px);
		}
		to {
			opacity: 1;
			transform: none;
		}
	}
	.th {
		margin-top: 0;
	}
	.card h3.eyebrow {
		margin: 0;
		font-size: 12px;
	}
	.stat .bg + div {
		background: linear-gradient(180deg, transparent 15%, var(--scrim)) !important;
	}
	.caret {
		display: inline-block;
		width: 3px;
		height: 1em;
		margin-left: 3px;
		vertical-align: -0.12em;
		background: currentColor;
		animation: blink 0.9s steps(1) infinite;
	}
	.type {
		display: grid;
	}
	.type > * {
		grid-area: 1/1;
	}
	.ghost {
		visibility: hidden;
	}
	.slide {
		position: absolute;
		inset: 0;
		opacity: 0;
		animation: kb 27s ease-in-out infinite;
	}
	.hero-slide {
		background-position: center;
		background-size: cover;
	}
	.hero-one {
		background-image:
			linear-gradient(180deg, rgba(7, 16, 38, 0.22), rgba(7, 16, 38, 0.56)), url('/coop/hero-1.jpg');
		background-position: center 42%;
	}
	.hero-two {
		background-image:
			linear-gradient(180deg, rgba(7, 16, 38, 0.22), rgba(7, 16, 38, 0.56)),
			url('/coop/hero-2.webp');
		background-position: center 42%;
	}
	.hero-three {
		background-image:
			linear-gradient(180deg, rgba(7, 16, 38, 0.22), rgba(7, 16, 38, 0.56)),
			url('/coop/hero-3.webp');
		background-position: center 42%;
	}
	.membership-backdrop {
		background-image:
			linear-gradient(180deg, rgba(7, 16, 38, 0.3), rgba(7, 16, 38, 0.48)),
			url('/coop/photo-membership.jpg');
		background-position:
			center,
			center 42%;
		background-size: cover;
	}
	.stat {
		position: relative;
		overflow: hidden;
		min-height: 210px;
		border-radius: 22px;
		border: 1px solid var(--line);
		display: flex;
		flex-direction: column;
		justify-content: flex-end;
		padding: 20px;
		color: #fff;
	}
	.stat .bg {
		position: absolute;
		inset: 0;
		animation: fade 8s ease-in-out infinite alternate;
	}
	.reel {
		display: inline-block;
		height: 56px;
		overflow: hidden;
		vertical-align: top;
	}
	.strip {
		display: flex;
		flex-direction: column;
		transition: transform 1.4s cubic-bezier(0.2, 1.35, 0.35, 1);
	}
	.strip span {
		display: block;
		height: 56px;
		line-height: 56px;
		text-align: center;
	}
	.row {
		display: flex;
		align-items: center;
		gap: 20px;
		padding: 22px 24px;
		border-radius: 20px;
		border: 1px solid var(--line);
		background: var(--card);
		color: var(--text);
		transition:
			transform 0.35s,
			border-color 0.35s;
	}
	.row:hover {
		transform: translateX(12px);
		border-color: var(--accent);
	}
	.row .go {
		font-size: 24px;
		color: var(--accent);
		transition: transform 0.35s;
	}
	.row:hover .go {
		transform: translateX(8px);
	}
	details {
		background: var(--card);
		border: 1px solid var(--line);
		border-radius: 16px;
		padding: 0 20px;
	}
	summary {
		display: flex;
		justify-content: space-between;
		gap: 16px;
		align-items: center;
		min-height: 60px;
		font-weight: 700;
		cursor: pointer;
		list-style: none;
	}
	summary::-webkit-details-marker {
		display: none;
	}
	summary::after {
		content: '+';
		font-size: 24px;
		color: var(--accent);
	}
	details[open] summary::after {
		content: '−';
	}
	details p {
		margin: 0 0 18px;
		color: var(--muted);
		line-height: 1.65;
	}
	.window {
		border-radius: 18px;
		overflow: hidden;
		background: #f4f6fb;
		color: #16213f;
		border: 1px solid var(--line);
		box-shadow: 0 30px 70px rgba(0, 0, 0, 0.35);
		animation: float 7s ease-in-out infinite;
	}
	.bar {
		display: flex;
		gap: 7px;
		align-items: center;
		padding: 12px 14px;
		background: #fff;
		border-bottom: 1px solid #e3e7f0;
		font-size: 12px;
		font-weight: 700;
	}
	.dot {
		width: 10px;
		height: 10px;
		border-radius: 50%;
	}
	.mini {
		background: #fff;
		border-radius: 12px;
		padding: 12px;
		font-size: 11px;
		color: #55607c;
	}
	.mini b {
		display: block;
		margin-top: 4px;
		font-size: 17px;
		color: #16213f;
	}
	.tx {
		display: flex;
		justify-content: space-between;
		gap: 8px;
		padding: 9px 0;
		border-top: 1px solid #eef0f5;
		font-size: 12px;
	}
	.why-layout {
		display: grid;
		grid-template-columns: minmax(0, 1.15fr) minmax(0, 0.85fr);
		gap: clamp(32px, 5vw, 64px);
		align-items: center;
	}
	.why-copy,
	.why-carousel {
		min-width: 0;
	}
	.why-pillars {
		grid-template-columns: repeat(auto-fit, minmax(min(100%, 220px), 1fr));
	}
	.why-carousel {
		width: 100%;
		max-width: 500px;
		justify-self: end;
	}
	.why-caption {
		margin: 4px 24px 0 0;
		color: var(--muted);
		text-align: center;
		font-size: 14px;
		line-height: 1.6;
	}
	.printing-intro {
		max-width: 760px;
		margin: 96px auto 0;
	}
	.deck {
		position: relative;
		height: clamp(340px, 42vw, 450px);
		touch-action: pan-y;
		user-select: none;
	}
	.photo {
		position: absolute;
		inset: 0 24px 30px 0;
		border: 0;
		padding: 24px;
		border-radius: 24px;
		display: flex;
		flex-direction: column;
		justify-content: flex-end;
		align-items: flex-start;
		text-align: left;
		color: #fff;
		background-position: center;
		background-size: cover;
		background-repeat: no-repeat;
		font:
			700 18px 'Plus Jakarta Sans',
			system-ui,
			sans-serif;
		box-shadow: 0 18px 40px rgba(0, 0, 0, 0.3);
		cursor: grab;
		transition:
			transform 0.45s cubic-bezier(0.2, 1, 0.3, 1),
			opacity 0.45s;
	}
	.photo-campus {
		background-image:
			linear-gradient(180deg, rgba(8, 15, 35, 0.04) 24%, rgba(8, 15, 35, 0.78)),
			url('/coop/coop_showcase-campus_life-4.jpg');
		background-position:
			center,
			center 42%;
	}
	.photo-members {
		background-image:
			linear-gradient(180deg, rgba(8, 15, 35, 0.02) 24%, rgba(8, 15, 35, 0.78)),
			url('/coop/coop_showcase-member_services-1.webp');
		background-position:
			center,
			center 40%;
	}
	.photo-community {
		background-image:
			linear-gradient(180deg, rgba(8, 15, 35, 0.02) 24%, rgba(8, 15, 35, 0.78)),
			url('/coop/coop_showcase-cooperative_community-2.jpg');
	}
	.service-image {
		display: block;
		width: 100%;
		height: 160px;
		margin-bottom: 16px;
		border-radius: 14px;
		object-fit: cover;
	}
	.zig {
		height: 12px;
		background:
			linear-gradient(135deg, #fffdf6 50%, transparent 50%) 0 0/16px 16px repeat-x,
			linear-gradient(225deg, #fffdf6 50%, transparent 50%) 0 0/16px 16px repeat-x;
	}
	.bars {
		height: 56px;
		background: repeating-linear-gradient(
			90deg,
			#111 0 3px,
			transparent 3px 5px,
			#111 5px 6px,
			transparent 6px 10px,
			#111 10px 15px,
			transparent 15px 17px
		);
	}
	.promo {
		position: relative;
		overflow: hidden;
		border-radius: 26px;
		padding: 30px;
		color: #fff;
		transition: transform 0.35s;
	}
	.promo:hover {
		transform: translateY(-6px) scale(1.01);
	}
	.orb {
		position: absolute;
		border-radius: 50%;
		background: rgba(255, 255, 255, 0.14);
		animation: float 9s ease-in-out infinite;
	}
	.stage {
		perspective: 1000px;
		padding: 14px 0;
	}
	.pass {
		border-radius: 22px;
		overflow: hidden;
		background: #fff;
		color: #111;
		box-shadow: 0 34px 60px rgba(0, 20, 80, 0.45);
		animation: passfloat 7s ease-in-out infinite;
	}
	.face {
		position: relative;
		overflow: hidden;
		padding: 24px;
		color: #fff;
		background: linear-gradient(140deg, #0766dd, #0543c9, #0634a8);
	}
	.face::after {
		content: '';
		position: absolute;
		inset: -40%;
		pointer-events: none;
		background: linear-gradient(
			115deg,
			transparent 42%,
			rgba(255, 255, 255, 0.22) 50%,
			transparent 58%
		);
		animation: sheen 5s ease-in-out infinite;
	}
	@keyframes passfloat {
		0%,
		100% {
			transform: translateY(0) rotate3d(1, -1, 0, -6deg);
		}
		50% {
			transform: translateY(-14px) rotate3d(1, -1, 0, 6deg);
		}
	}
	@keyframes sheen {
		0%,
		15% {
			transform: translateX(-70%);
		}
		75%,
		100% {
			transform: translateX(70%);
		}
	}
	@keyframes shine {
		0%,
		100% {
			background-position: 0 0;
		}
		50% {
			background-position: 100% 0;
		}
	}
	@keyframes blink {
		50% {
			opacity: 0;
		}
	}
	@keyframes kb {
		0% {
			opacity: 0;
			transform: scale(1);
		}
		6% {
			opacity: 1;
		}
		33% {
			opacity: 1;
		}
		41% {
			opacity: 0;
			transform: scale(1.07);
		}
		100% {
			opacity: 0;
			transform: scale(1);
		}
	}
	@keyframes fade {
		0%,
		35% {
			opacity: 0;
		}
		65%,
		100% {
			opacity: 1;
		}
	}
	@keyframes float {
		50% {
			transform: translateY(-10px);
		}
	}
	@keyframes pulse {
		50% {
			box-shadow: 0 0 0 14px rgba(255, 208, 74, 0);
		}
	}
	@keyframes slidein {
		from {
			opacity: 0;
			transform: translateX(-60px);
		}
		to {
			opacity: 1;
			transform: none;
		}
	}
	@keyframes up {
		from {
			opacity: 0;
			transform: translateY(40px);
		}
		to {
			opacity: 1;
			transform: none;
		}
	}
	@keyframes para {
		from {
			transform: translateY(-40px);
		}
		to {
			transform: translateY(40px);
		}
	}
	@supports not (animation-timeline: view()) {
		.h2:not(.th),
		.fu {
			animation: up 0.9s ease both;
		}
	}
	@supports (animation-timeline: view()) {
		.h2:not(.th),
		.fu {
			animation: up linear both;
			animation-timeline: view();
			animation-range: entry 0% cover 24%;
		}
		.reveal {
			animation: up linear both;
			animation-timeline: view();
			animation-range: entry 0% entry 60%;
		}
		.slidein {
			animation: slidein linear both;
			animation-timeline: view();
			animation-range: entry 0% entry 80%;
		}
		.para {
			animation: para linear both;
			animation-timeline: view();
		}
	}
	@media (prefers-reduced-motion: reduce) {
		*,
		*::after {
			animation: none !important;
			transition: none !important;
		}
		.slide:first-child {
			opacity: 1;
		}
	}

	@supports not (animation-timeline: view()) {
		h1,
		h2,
		h3 {
			animation: up 0.9s ease both;
		}
	}
	@supports (animation-timeline: view()) {
		h1,
		h2,
		h3 {
			animation: up linear both;
			animation-timeline: view();
			animation-range: entry 0% cover 24%;
		}
	}
	:global(.typewriter-char) {
		display: inline-block;
		white-space: pre;
		opacity: 0;
		transition: opacity 0.08s linear;
	}
	:global(.typewriter-char.is-visible) {
		opacity: 1;
	}
	@media (prefers-reduced-motion: reduce) {
		:global(.typewriter-char) {
			opacity: 1 !important;
		}
	}
	@media (max-width: 860px) {
		.why-layout {
			grid-template-columns: minmax(0, 1fr);
		}
		.why-carousel {
			max-width: 560px;
			justify-self: center;
		}
	}
	.brand-mark,
	.footer-brand-mark,
	.loyalty-mark {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		flex-shrink: 0;
		font-weight: 900;
		letter-spacing: -0.06em;
	}
	.brand-mark {
		width: 66px;
		height: 36px;
		border-radius: 999px;
		background: #fff;
		color: #10214a;
		font-size: 15px;
	}
	.footer-brand-mark {
		width: 120px;
		height: 65px;
		border-radius: 8px;
		background: linear-gradient(135deg, #0b1330, #e9b52c);
		color: #fff;
		font-size: 28px;
	}
	.loyalty-mark {
		width: 51px;
		height: 51px;
		border-radius: 13px;
		background: linear-gradient(135deg, #eabf35, #f8e7a5);
		color: #17203f;
		font-size: 17px;
	}
</style>
