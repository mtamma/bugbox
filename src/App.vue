<template>
	<div id="bugbox-app" :class="classes">
		<div ref="cover" class="bugbox__cover">
			<div ref="screenshotArea" class="bugbox__screenshot-area">
				<span>Hold Ctrl and use MouseWheel to resize screenshot area</span>
			</div>

			<pins />
		</div>

		<div :class="panelClasses">
			<iframe ref="panelFrame"></iframe>

			<panel :frame="panelFrame" />
		</div>

		<div class="bugbox__issue-form">
			<iframe ref="issueFrame"></iframe>

			<issue-form :frame="issueFrame" />
		</div>
	</div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex';
import { getDocumentHeight } from 'helpers/utils';
import Pins from 'components/Pins.vue';
import Panel from 'components/Panel.vue';
import IssueForm from 'components/IssueForm.vue';

export default {
	name: 'bugbox',

	components: {
		Panel,
		Pins,
		IssueForm
	},

	data () {
		return {
			panelFrame: null,
			issueFrame: null,
			autoResizeInterval: null
		};
	},

	computed: {
		...mapGetters([
			'panelCollapsed',
			'tagged',
			'project',
			'status',
			'isSubmittingIssue'
		]),

		/**
		 * Get classes object.
		 * @return {Object}
		 */
		classes() {
			return [
				'bugbox',
				{ 'bugbox--tagging': this.status === 'tagging' },
				{ 'bugbox--tagged': this.tagged }
			];
		},

		/**
		 * Get panel classes object.
		 * @return {Object}
		 */
		panelClasses() {
			return [
				'bugbox__panel',
				{ 'bugbox__panel--collapsed': this.panelCollapsed }
			];
		},

		/**
		 * Get issue id from window location.
		 * @return {String}
		 */
		issueFromLocation() {
			return (window.location.href.match(/\#issue\-(.+)$/) || [])[1];
		},
	},

	methods: {
		...mapActions([
			'init',
			'initTagManager',
			'initTagging',
			'setTempPin',
			'setTagged',
			'selectIssue',
			'updateCurrentUrl'
		]),

		/**
		 * Bind hotkeys.
		 * @return {void}
		 */
		bindHotkeys() {
			window.addEventListener('keydown', this.handleKeydown);
		},

		/**
		 * Unbind hotkeys.
		 * @return {[type]} [description]
		 */
		unbindHotkeys() {
			window.removeEventListener('keydown', this.handleKeydown);
		},

		/**
		 * Handle keydown event.
		 * @param  {[type]} event [description]
		 * @return {[type]}       [description]
		 */
		handleKeydown(event) {
			/**
			 * Init tagging with Ctrl+Shif+A
			 */
			if (
				this.project &&
				!this.isSubmittingIssue &&
				event.ctrlKey &&
				event.shiftKey &&
				event.keyCode === 65
			) {
				event.preventDefault();
				event.stopPropagation();
				this.initTagging();
			}
		},

		/**
		 * Highlight issue from window location.
		 * @param  {Object} project
		 * @return {void}
		 */
		highlightIssueFromLocation(project) {
			if (this.issueFromLocation && project && project.issues) {
				const issue = project.issues.find(issue => {
					if (!issue.attachments || !issue.attachments.length) {
						return false;
					}

					return issue.attachments.some(attachment => attachment.url.indexOf(this.issueFromLocation) >= 0);
				});

				if (issue) {
					return this.selectIssue(issue.id);
				}
			}
		},

		/**
		 * Setup TagManager instance.
		 * @return {[type]} [description]
		 */
		setupTagManager() {
			this.initTagManager({
				cover         : this.$refs.cover,
				screenshotArea: this.$refs.screenshotArea,
				initTagging   : this.initTagging,
				onBeforeTagged: this.setTempPin,
				onTagged      : this.setTagged
			});
		},

		/**
		 * Set auto resize interval.
		 * @return {[type]} [description]
		 */
		autoResizeCover() {
			this.autoResizeInterval = setInterval(() => {
				this.$refs.cover.style.height = '';
				this.$refs.cover.style.height = getDocumentHeight() + 'px';
			}, 100)
		},

		/**
		 * Handle hash change event.
		 * @param  {Event} event
		 * @return {void}
		 */
		handleHashChange(event) {
			this.updateCurrentUrl();
		}
	},

	created() {
		this.bindHotkeys();
		this.init()
			.then(this.highlightIssueFromLocation);
	},

	mounted() {
		this.panelFrame = this.$refs.panelFrame;
		this.issueFrame = this.$refs.issueFrame;

		this.setupTagManager();
		this.autoResizeCover();

		window.addEventListener('hashchange', this.handleHashChange);
	},

	beforeDestroy() {
		this.unbindHotkeys();
		clearInterval(this.autoResizeInterval);

		window.removeEventListener('hashchange', this.handleHashChange);
	}
}
</script>

<style>
	/*  Body  */
	.bugbox,
	.bugbox * { all: initial; padding: 0; margin: 0; outline: 0; box-sizing: border-box; }

	.bugbox iframe { border: 0; padding: 0; margin: 0; width: 100%; height: 100%; }

	/*  Cover  */
	.bugbox .bugbox__cover { position: absolute; z-index: 2147483647; left: 0; top: 0; width: 100%; min-height: 100%; background: rgba(255,255,255,0); pointer-events: none; visibility: hidden; overflow: hidden; }
	.bugbox .bugbox__cover { cursor: url(~assets/images/cursor.png) 15 40, pointer; }

	/*  Screenshot Area  */
	.bugbox .bugbox__screenshot-area { position: fixed; z-index: 2147483647; left: 0; top: 0; width: 100%; height: 100%; background-clip: content-box; border-style: solid; border-color: rgba(0,0,0,.5); border-width: 0; visibility: hidden; pointer-events: none; transition: border-width .2s; }
	.bugbox .bugbox__screenshot-area span { position: absolute; left: 0; top: 100%; max-width: 100%; padding: 2px 0; color: #fff; font-family: sans-serif; font-size: 10px; white-space: nowrap; visibility: hidden; pointer-events: none; }

	/*  Panel  */
	.bugbox .bugbox__panel { position: fixed; z-index: 2147483647; right: 0; bottom: 0; width: 340px; height: 100%; transition: transform .2s, width .2s ; }
	.bugbox .bugbox__panel--collapsed { width: 40px; height: 120px; transition: transform .2s, width .2s, height .2s .2s; }

	/*  Pins  */
	.bugbox .bugbox__pins { transition: all .2s; }

	/*  Issue Form  */
	.bugbox .bugbox__issue-form { position: fixed; z-index: 2147483647; left: 50%; top: 50%; width: 660px; height: 550px; max-width: 100vw; max-height: 100vh; transform: translate(-50%, -50%); visibility: hidden; }
	.bugbox .bugbox__issue-form iframe { pointer-events: none; }

	/*  Tagged State  */
	.bugbox--tagged .bugbox__panel { box-shadow: none; transform: translateX(100%); }
	.bugbox--tagged .bugbox__issue-form { visibility: visible; display: block; }
	.bugbox--tagged .bugbox__issue-form iframe { pointer-events: all; }

	/*  Tagging State  */
	.bugbox--tagging .bugbox__cover { visibility: visible; pointer-events: all; }
	.bugbox--tagging .bugbox__screenshot-area,
	.bugbox--tagging .bugbox__screenshot-area span { visibility: visible; }
	.bugbox--tagging .bugbox__panel { box-shadow: none; transform: translateX(100%); }
	.bugbox--tagging .bugbox__issue-form { display: none; }
</style>
