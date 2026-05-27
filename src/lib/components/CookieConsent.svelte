<script lang="ts">
	import { onMount } from 'svelte';
	import { Button } from '$lib/components/ui/button';
	import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '$lib/components/ui/card';
	import { Checkbox } from '$lib/components/ui/checkbox';
	import * as Dialog from '$lib/components/ui/dialog';
	import Icon from '@iconify/svelte';

	interface ConsentPreferences {
		necessary: boolean;
		functional: boolean;
		analytics: boolean;
	}

	let showBanner = $state(false);
	let showSettings = $state(false);

	let preferences = $state<ConsentPreferences>({
		necessary: true,
		functional: false,
		analytics: false
	});

	let agreed = $state(false);
	let overlayContent = $state<string | null>(null);
	let readAgreement = $state(false);
	let readPrivacy = $state(false);
	let canAgree = $derived(readAgreement && readPrivacy);

	const STORAGE_KEY = 'cookie-consent-preferences';
	const CONSENT_VERSION = '2.0';

	onMount(() => {
		loadPreferences();

		const handleClick = (e: Event) => {
			const target = e.target as HTMLElement;
			if (target.id === 'open_preferences_center' || target.closest('#open_preferences_center')) {
				e.preventDefault();
				showSettings = true;
			}
		};

		document.addEventListener('click', handleClick);
		return () => {
			document.removeEventListener('click', handleClick);
		};
	});

	function loadPreferences() {
		try {
			const stored = localStorage.getItem(STORAGE_KEY);
			if (stored) {
				const data = JSON.parse(stored);
				if (data.version === CONSENT_VERSION && data.agreed) {
					preferences = data.preferences;
					agreed = true;
					applyConsent();
					return;
				}
			}
		} catch (e) {
			console.error('Failed to load cookie preferences:', e);
		}

		showBanner = true;
	}

	function savePreferences(close = true) {
		try {
			localStorage.setItem(STORAGE_KEY, JSON.stringify({
				version: CONSENT_VERSION,
				preferences,
				agreed: true,
				timestamp: Date.now()
			}));
			agreed = true;
		} catch (e) {
			console.error('Failed to save cookie preferences:', e);
		}
		if (close) {
			applyConsent();
			showBanner = false;
			showSettings = false;
		}
	}

	function acceptAll() {
		preferences = {
			necessary: true,
			functional: true,
			analytics: true
		};
		savePreferences();
	}

	function acceptNecessary() {
		preferences = {
			necessary: true,
			functional: false,
			analytics: false
		};
		savePreferences();
	}

	function saveCustomPreferences() {
		preferences.necessary = true;
		savePreferences();
	}

	function withdrawConsent() {
		try {
			localStorage.removeItem(STORAGE_KEY);
		} catch {}
		agreed = false;
		readAgreement = false;
		readPrivacy = false;
		showBanner = true;
		showSettings = false;
	}

	function applyConsent() {
		window.dispatchEvent(new CustomEvent('cookie-consent-updated', {
			detail: preferences
		}));
	}
</script>

<!-- Cookie 横幅 -->
{#if showBanner}
	<div class="fixed inset-0 z-50 bg-background/80 backdrop-blur-sm">
		<div class="fixed bottom-0 left-0 right-0 p-4 md:p-6">
			<Card class="mx-auto max-w-3xl">
				<CardHeader class="pb-2">
					<CardTitle class="flex items-center gap-2">
						<Icon icon="mdi:cookie" class="h-5 w-5" />
						隐私与协议
					</CardTitle>
				</CardHeader>
				<CardContent class="pt-0">
					<p class="text-xs text-muted-foreground mb-3">
						继续使用本网站即表示你同意以下协议及隐私政策中所述的 Cookie 使用方式。
					</p>
					<div class="space-y-4">
						<p class="text-sm text-muted-foreground">
							点击"接受全部"即表示您同意我们使用所有 Cookie，您也可以点击"自定义设置"来选择您希望启用的 Cookie 类型。
						</p>

			<label class="flex items-start gap-3 p-3 rounded-lg border bg-muted/30">
				<Checkbox bind:checked={agreed} disabled={!canAgree} class="mt-0.5" />
				<div class="text-xs space-y-1">
					<span>我已阅读并同意</span>
					<button type="button" class="text-primary underline hover:text-primary/80" onclick={() => overlayContent = 'agreement'}>《用户协议》</button>
					<span>和</span>
				<button type="button" class="text-primary underline hover:text-primary/80" onclick={() => overlayContent = 'privacy'}>《隐私政策》</button>
			</div>
			{#if !canAgree}
				<div class="text-[10px] text-muted-foreground mt-1">请先点击上方链接阅读并确认同意协议</div>
			{/if}
		</label>

						<div class="flex flex-wrap gap-3">
							<Button onclick={acceptAll} disabled={!agreed}>
								<Icon icon="mdi:check-all" class="mr-2 h-4 w-4" />
								接受全部
							</Button>
							<Button variant="outline" onclick={acceptNecessary} disabled={!agreed}>
								仅必要 Cookie
							</Button>
							<Button variant="outline" onclick={() => showSettings = true} disabled={!agreed}>
								<Icon icon="mdi:cog" class="mr-2 h-4 w-4" />
								自定义设置
							</Button>
						</div>
					</div>
				</CardContent>
			</Card>
		</div>
	</div>
{/if}

<!-- Cookie 设置对话框 -->
<Dialog.Root bind:open={showSettings}>
	<Dialog.Content class="max-w-2xl max-h-[90vh] overflow-y-auto">
		<Dialog.Header>
			<Dialog.Title>隐私与协议设置</Dialog.Title>
			<Dialog.Description>
				请阅读并同意用户协议与隐私政策，并选择您希望启用的 Cookie 类型。必要 Cookie 无法禁用。
			</Dialog.Description>
		</Dialog.Header>

		<div class="space-y-6 py-4">
						<label class="flex items-start gap-3 p-3 rounded-lg border bg-muted/30">
							<Checkbox bind:checked={agreed} disabled={!canAgree} class="mt-0.5" />
							<div class="text-xs space-y-1">
								<span>我已阅读并同意</span>
								<button type="button" class="text-primary underline hover:text-primary/80" onclick={() => overlayContent = 'agreement'}>《用户协议》</button>
								<span>和</span>
								<button type="button" class="text-primary underline hover:text-primary/80" onclick={() => overlayContent = 'privacy'}>《隐私政策》</button>
							</div>
							{#if !canAgree}
								<div class="text-[10px] text-muted-foreground mt-1">请先点击上方链接阅读并确认同意协议</div>
							{/if}
						</label>

			<!-- 必要 Cookie -->
			<div class="space-y-3">
				<div class="flex items-start gap-3">
					<Checkbox checked={true} disabled={true} class="mt-1" />
					<div class="flex-1">
						<h3 class="font-semibold">必要 Cookie</h3>
						<p class="text-sm text-muted-foreground mt-1 mb-2">
							这些 Cookie 对于网站的基本功能是必需的，无法禁用。
						</p>
						<ul class="text-sm text-muted-foreground space-y-1 list-disc list-inside">
							<li>Umami Analytics - 网站统计</li>
							<li>Cloudflare Insights - 性能监控</li>
						</ul>
					</div>
				</div>
			</div>

			<!-- 功能性 Cookie -->
			<div class="space-y-3">
				<div class="flex items-start gap-3">
					<Checkbox bind:checked={preferences.functional} class="mt-1" />
					<div class="flex-1">
						<h3 class="font-semibold">功能性 Cookie</h3>
						<p class="text-sm text-muted-foreground mt-1 mb-2">
							这些 Cookie 用于增强网站功能和个性化体验。
						</p>
						<ul class="text-sm text-muted-foreground space-y-1 list-disc list-inside">
							<li>Giscus - 评论系统</li>
						</ul>
					</div>
				</div>
			</div>

			<!-- 分析 Cookie -->
			<div class="space-y-3">
				<div class="flex items-start gap-3">
					<Checkbox bind:checked={preferences.analytics} class="mt-1" />
					<div class="flex-1">
						<h3 class="font-semibold">分析 Cookie</h3>
						<p class="text-sm text-muted-foreground mt-1 mb-2">
							这些 Cookie 帮助我们了解访问者如何使用网站，以便改进用户体验。
						</p>
						<ul class="text-sm text-muted-foreground space-y-1 list-disc list-inside">
							<li>百度统计 - 访问分析</li>
							<li>Google Analytics - 用户行为分析</li>
							<li>Microsoft Clarity - 用户体验分析</li>
						</ul>
					</div>
				</div>
			</div>
		</div>

		<Dialog.Footer class="flex-wrap gap-2">
			<Button variant="ghost" size="sm" onclick={withdrawConsent} class="text-destructive hover:text-destructive">
				<Icon icon="mdi:close-circle-outline" class="size-3.5 mr-1" />撤回同意
			</Button>
			<div class="flex-1" />
			<Button variant="outline" onclick={acceptNecessary} disabled={!agreed}>
				仅必要 Cookie
			</Button>
			<Button onclick={saveCustomPreferences} disabled={!agreed}>
				保存设置
			</Button>
			<Button onclick={acceptAll} disabled={!agreed}>
				接受全部
			</Button>
		</Dialog.Footer>
	</Dialog.Content>
</Dialog.Root>

<!-- 全屏协议展示 -->
{#if overlayContent}
	<div class="fixed inset-0 z-50 bg-background overflow-y-auto" onclick={() => overlayContent = null}>
		<div class="max-w-2xl mx-auto px-6 py-8" onclick={(e) => e.stopPropagation()}>
			<div class="flex items-center justify-between mb-4">
				<h2 class="text-lg font-bold">{overlayContent === 'agreement' ? '用户协议' : '隐私政策'}</h2>
				<button type="button" class="text-muted-foreground hover:text-foreground" onclick={() => overlayContent = null}>
					<Icon icon="mdi:close" class="size-6" />
				</button>
			</div>
			<div class="text-sm space-y-3 leading-relaxed">
				{#if overlayContent === 'agreement'}
					<div class="space-y-4">
						<section>
							<h3 class="font-bold text-base mb-2">免责声明</h3>
							<p>本网站（以下简称"本站"）是一个个人项目，按「现状」及「可用」基础提供。在法律允许的最大范围内，本站明确声明不承担任何明示或暗示的担保责任，包括但不限于适销性、特定用途适用性及不侵权的担保。本站不保证服务的连续性、及时性、安全性及准确性，你使用本站服务所产生的全部风险由你自行承担。</p>
							<p>你明确理解并同意，本站运营者不对因使用或无法使用本站服务所导致的任何直接、间接、偶然、特殊及后续的损害承担责任，包括但不限于利润损失、数据丢失、业务中断、声誉损害及其他商业损失。</p>
							<p>本站所有生成内容由人工智能自动生成，不代表本站运营者的观点、立场或意见。生成内容的准确性、完整性、合法性及实用性本站不作任何保证。你应对你生成、发布及传播的内容承担全部责任。</p>
						</section>

						<section>
							<h3 class="font-bold text-base mb-2">论坛使用规则（/forum）</h3>
							<ol class="list-decimal list-inside space-y-1">
								<li>禁止发布任何违反中华人民共和国法律法规的内容，包括但不限于危害国家安全、煽动民族仇恨、破坏国家统一、宣扬恐怖主义、传播淫秽色情、赌博、暴力、凶杀、恐怖或者教唆犯罪的信息。</li>
								<li>禁止发布任何侵犯他人合法权益的内容，包括但不限于侵犯他人名誉权、肖像权、知识产权、隐私权及商业秘密。</li>
								<li>禁止恶意攻击、辱骂、骚扰、威胁、歧视其他用户。论坛讨论应保持基本网络礼仪，理性表达观点。</li>
								<li>禁止发布垃圾广告、恶意推广、刷屏、灌水及一切形式的垃圾信息。</li>
								<li>禁止利用论坛进行任何形式的网络诈骗、钓鱼、传播恶意软件或链接。</li>
								<li>禁止绕越或试图绕越论坛的审核、封禁等管理措施。</li>
								<li>论坛管理员有权在不另行通知的情况下删除违规内容、限制或封禁违规账号。</li>
							</ol>
						</section>

						<section>
							<h3 class="font-bold text-base mb-2">AI 生图服务使用规则（/draw）</h3>
							<ol class="list-decimal list-inside space-y-1">
								<li>禁止生成、上传或传播任何涉及儿童性剥削（CSAM）、非自愿色情内容及一切形式的违法色情内容。</li>
								<li>禁止生成、上传或传播任何包含仇恨言论、极端暴力、恐怖主义、歧视性内容（包括但不限于种族、民族、宗教、性别、性取向、残疾歧视）。</li>
								<li>禁止利用本服务生成、伪造或传播虚假信息、误导性内容及深度伪造内容。</li>
								<li>禁止利用本服务侵犯他人知识产权，包括但不限于未经授权使用受版权保护的图像、角色及商标。</li>
								<li>禁止反向工程、破解、攻击或以任何方式破坏本服务的后端系统、队列机制及点券系统。</li>
								<li>禁止批量注册、自动化脚本及一切形式的滥用行为。</li>
								<li>本站有权审查你提交的提示词及生成的内容。若发现违规，我们保留限制或永久封禁账号的权利，恕不另行通知。</li>
								<li>你同意本站阅读、审查、存档、分析及分享你生成的所有内容。即使你在账号界面中删除相关记录，本站仍可能在后端保留副本，用于审核、统计、模型改进及其他合法用途。</li>
							</ol>
						</section>

						<section>
							<h3 class="font-bold text-base mb-2">点券及内购服务</h3>
							<ol class="list-decimal list-inside space-y-1">
								<li>站内点券为虚拟商品，一经充值售出，概不退换、不折现、不延期。</li>
								<li>本站保留随时调整点券定价、消耗规则及服务内容的权利，调整后的规则自发布之日起生效。</li>
								<li>因违规被封禁的账号，剩余点券不予退还。</li>
							</ol>
						</section>

						<section>
							<h3 class="font-bold text-base mb-2">其他条款</h3>
							<ol class="list-decimal list-inside space-y-1">
								<li>本站保留随时修改本协议的权利，修改后的协议自发布之日起生效。你继续使用本站服务即视为接受修改后的协议。</li>
								<li>本协议适用中华人民共和国法律，并按其解释。因本协议引起的或与本协议有关的争议，双方应友好协商解决；协商不成的，提交本站运营者所在地有管辖权的人民法院诉讼解决。</li>
								<li>如本协议的任何条款被认定为无效或不可执行，其余条款仍应保持完全效力。</li>
								<li>若你对本协议有任何疑问，可通过本站提供的联系方式与运营者取得联系。</li>
							</ol>
						</section>
					</div>
				{:else}
					<p><strong>必要服务（始终加载）：</strong>Umami Analytics（自托管）收集匿名访问数据；Cloudflare Web Analytics 无 Cookie 无指纹追踪。</p>
					<p><strong>广告：</strong>Google Adsense 展示广告，使用 Cookie 提供个性化广告。</p>
					<p><strong>功能（需同意）：</strong>Giscus 基于 GitHub Discussions 的评论系统。</p>
					<p><strong>分析（需同意）：</strong>百度统计、Google Analytics、Microsoft Clarity 收集站点访问与用户行为数据。</p>
					<p><strong>本地存储：</strong>cookie-consent-preferences（同意偏好）、theme（主题）、论坛登录凭证。</p>
					<p><strong>其他：</strong>Cloudflare Turnstile 人机验证；按需加载 CDN 资源。</p>
					<p>数据控制：可通过本页面底部的「隐私与协议设置」按钮随时修改。清除 localStorage 可删除本地存储的所有数据。</p>
				{/if}
			</div>
			<div class="mt-6 flex justify-center">
				<button type="button" class="px-6 py-2 rounded-lg bg-primary text-primary-foreground text-sm font-medium hover:bg-primary/90" onclick={() => { if (overlayContent === 'agreement') readAgreement = true; if (overlayContent === 'privacy') readPrivacy = true; overlayContent = null; }}>
					我已阅读
				</button>
			</div>
		</div>
	</div>
{/if}
