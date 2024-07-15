<template>
	<header :class="{ 'scrolled-nav': scrollNav }">
		<nav>
			<div class="branding">
				<img src="../assets/logo.png" alt="img" />
			</div>

			<!-- Navbar -->
			<div class="nav-div">
				<ul v-show="!mobile" class="navigations">
					<li>
						<router-link
							class="link"
							to="/home"
							@click.native="setActiveMenuItemAndCloseNav('home')"
						>
							Home
						</router-link>
					</li>

					<li>
						<router-link
							class="link"
							to="/dashboard"
							@click.native="setActiveMenuItemAndCloseNav('dashboard')"
						>
							Dashboard
						</router-link>
					</li>

					<li>
						<router-link
							class="link"
							to="/projects"
							@click.native="setActiveMenuItemAndCloseNav('projects')"
						>
							Projects
						</router-link>
					</li>

					<template v-if="isLoggedIn">
						<li>
							<router-link
								class="link"
								to="/submit"
								@click.native="setActiveMenuItemAndCloseNav('submit')"
							>
								Submit PR
							</router-link>
						</li>

						<li>
							<router-link
								class="link"
								to="/myPR"
								@click.native="setActiveMenuItemAndCloseNav('myPR')"
							>
								My PR
							</router-link>
						</li>

						<div class="animation start-home"></div>
					</template>
				</ul>
			</div>

			<div class="auth-div">
				<ul v-show="!mobile" class="authnavigations">
					<template v-if="!isLoggedIn">
						<li>
							<router-link class="auth" to="/auth"> Login </router-link>
						</li>

						<li>
							<router-link class="auth" to="/signup"> Sign Up </router-link>
						</li>
					</template>

					<template v-else>
						<li @click="logout" class="auth">Log Out</li>
					</template>
				</ul>
			</div>

			<div class="icon" @click="toggleMobileNav">
				<i
					v-show="mobile && !mobileNav"
					class="fas fa-bars"
					:class="{ 'icon-active': mobileNav }"
					style="
						font-size: 1.5rem;
						position: fixed;
						top: 20px;
						right: 25px;
						cursor: pointer;
						z-index: 100;
					"
				></i>
			</div>

			<div class="icon" @click="toggleMobileNav">
				<i
					v-show="mobile && mobileNav"
					class="fas fa-times"
					:class="{ 'icon-active': mobileNav }"
					style="
						font-size: 1.5rem;
						position: fixed;
						top: 20px;
						right: 25px;
						cursor: pointer;
						z-index: 100;
					"
				></i>
			</div>

			<transition name="mobile-nav">
				<ul v-show="mobileNav" class="dropdown-nav">
					<li>
						<router-link
							class="link"
							:class="{ active: activeMenuItem === 'home' }"
							to="/home"
							@click.native="setActiveMenuItemAndCloseNav('home')"
						>
							Home
						</router-link>
					</li>

					<li>
						<router-link
							class="link"
							:class="{ active: activeMenuItem === 'dashboard' }"
							to="/dashboard"
							@click.native="setActiveMenuItemAndCloseNav('dashboard')"
						>
							Dashboard
						</router-link>
					</li>

					<li>
						<router-link
							class="link"
							:class="{ active: activeMenuItem === 'projects' }"
							to="/projects"
							@click.native="setActiveMenuItemAndCloseNav('projects')"
						>
							Projects
						</router-link>
					</li>

					<template v-if="isLoggedIn">
						<li>
							<router-link
								class="link"
								:class="{ active: activeMenuItem === 'myPR' }"
								to="/mypr"
								@click.native="setActiveMenuItemAndCloseNav('myPR')"
							>
								My PR
							</router-link>
						</li>

						<li @click="logout" class="link">Log out</li>
					</template>

					<template v-else>
						<li>
							<router-link
								class="link"
								:class="{ active: activeMenuItem === 'signup' }"
								to="/signup"
								@click.native="setActiveMenuItemAndCloseNav('signup')"
							>
								Sign Up
							</router-link>
						</li>
						<li>
							<router-link
								class="link"
								:class="{ active: activeMenuItem === 'auth' }"
								to="/login"
								@click.native="toggleLogin()"
							>
								Login
							</router-link>
						</li>
					</template>
				</ul>
			</transition>
		</nav>
	</header>
	<Login
		v-if="showLogin"
		:toggle="toggleLogin"
		@loginSuccess="handleLoginSuccess"
	/>
	<Signup
		v-if="showSignup"
		:toggle="toggleSignup"
		@signupSuccess="handleSignupSuccess"
	/>
</template>

<script>
import { projectAuth } from '../firebase/config'
import Login from '@/components/Login'
import Signup from '@/components/Signup'
import { successToast, errorToast } from '../composables/useToast'

export default {
	name: 'Nav',
	components: {
		Login,
		Signup,
	},
	data() {
		return {
			scrollNav: false,
			mobile: false,
			mobileNav: false,
			windowWidth: null,
			isLoggedIn: false,
			showLogin: false,
			showSignup: false,
			activeMenuItem: '',
		}
	},
	created() {
		window.addEventListener('resize', this.checkScreen)
		this.checkScreen()
		this.checkUserLoggedIn()
	},
	mounted() {
		window.addEventListener('scroll', this.updateScroll)
	},
	methods: {
		toggleMobileNav() {
			this.mobileNav = !this.mobileNav
		},
		closeMobileNav() {
			this.mobileNav = false
		},
		toggleLogin() {
			this.showLogin = !this.showLogin
		},
		toggleSignup() {
			this.showSignup = !this.showSignup
		},
		checkScreen() {
			this.windowWidth = window.innerWidth
			if (this.windowWidth <= 850) {
				this.mobile = true
			} else {
				this.mobile = false
				this.mobileNav = false
			}
		},
		updateScroll() {
			const scrollPosition = window.scrollY
			this.scrollNav = scrollPosition > 50
		},
		async checkUserLoggedIn() {
			projectAuth.onAuthStateChanged((user) => {
				this.isLoggedIn = !!user
			})
		},
		async logout() {
			try {
				await projectAuth.signOut()
				this.isLoggedIn = false
				successToast('Success', 'Logout successful')
			} catch (error) {
				errorToast('Error', 'Logout failed')
				console.error('Logout error:', error.message)
			}
		},
		setActiveMenuItemAndCloseNav(item) {
			this.activeMenuItem = item
			this.mobileNav = false
		},
		handleLoginSuccess() {
			successToast('Success', 'Login successful')
			this.showLogin = false
		},
		handleSignupSuccess() {
			successToast('Success', 'Signup successful')
			this.showSignup = false
		},
	},
}
</script>

<style scoped>
header {
	z-index: 99;
	position: fixed;
	width: 100vw;
	transition: 0.5s ease all;
	color: white;
}

nav {
	display: flex;
	width: 100vw;
	box-sizing: border-box;
	margin-top: 0.6rem;

	ul,
	.link {
		font-weight: 500;
		color: white;
		list-style: none;
		align-items: center;
		text-align: center;
		text-decoration: none;
	}

	li {
		padding: 0.8rem;

		width: 10rem;
		z-index: 100;
	}

	.link {
		font-size: 1.1rem;
		transition: 0.5s ease all;
		padding-bottom: 0.2rem;
		border-bottom: 0.1rem solid transparent;
		cursor: pointer;
		width: 10rem;
		text-align: center;

		&:hover {
			color: black;
			z-index: 100;
		}
	}

	.branding {
		display: flex;
		justify-content: center;
		align-items: center;
		width: 10vw;

		img {
			width: 3rem;
			transition: 0.5s ease all;
		}
	}
	.nav-div {
		position: relative;
		width: 70vw;
		padding-left: 4rem;
		justify-content: center;
		align-items: center;
	}

	.navigations {
		position: relative;
		display: flex;
		justify-content: center;
		align-items: center;
		list-style-type: none;
	}

	.navigations .animation {
		position: fixed;
		height: 3.5rem;
		top: 7px;
		z-index: 0;
		background-color: #0891b3;
		border-radius: 0.5rem;
		transition: all 0.5s ease 0s;
	}

	.navigations li:nth-child(1) {
		width: 7rem;
	}
	.navigations .start-home,
	li:nth-child(1):hover ~ .animation {
		width: 7rem;
		left: 24.7rem;
	}
	.navigations li:nth-child(2) {
		width: 9rem;
	}
	.navigations li:nth-child(2):hover ~ .animation {
		width: 9rem;
		left: 31.76rem;
	}
	.navigations li:nth-child(3) {
		width: 8rem;
	}
	.navigations li:nth-child(3):hover ~ .animation {
		width: 8rem;
		left: 40.7rem;
	}
	.navigations li:nth-child(4) {
		width: 9rem;
	}
	.navigations li:nth-child(4):hover ~ .animation {
		width: 9rem;
		left: 48.6rem;
	}
	.navigations li:nth-child(5) {
		width: 7rem;
	}
	.navigations li:nth-child(5):hover ~ .animation {
		width: 7rem;
		left: 57.6rem;
	}

	.auth-div {
		width: 20%;
		padding-left: 3rem;
	}
	.authnavigations {
		display: flex;
	}

	.icon {
		position: absolute;
		left: 50px;
		display: flex;
		align-items: center;
		height: 100%;
		cursor: pointer;
		font-size: 1.5rem;
		transition: 0.5s ease all;
	}

	.icon-active {
		transform: rotate(180deg);
	}

	.dropdown-nav {
		display: flex;
		flex-direction: column;
		position: fixed;
		width: 100%;
		max-width: 15.62rem;
		height: 100%;
		background-color: #18181b;
		top: 0;
		right: 0;
		padding-top: 3.75rem;
	}

	.dropdown-nav .link.active {
		display: block;
		background-color: #0891b3;

		border-radius: 1.25rem;

		text-align: center;
		color: white;
		height: 2.68rem;
		width: 100%;
		transition: background-color 0.7s ease;
	}

	.dropdown-nav .link.active:hover {
		background-color: #0891b3;
	}

	.mobile-nav-enter-active,
	.mobile-nav-leave-active {
		transition: 1s ease all;
	}

	.mobile-nav-enter-from,
	.mobile-nav-leave-to {
		transform: translateX(15.625rem);
	}

	.mobile-nav-enter-to {
		transform: translateX(0);
	}

	.auth {
		color: #0891b3;
		border: #0891b3 1px solid;
		border-radius: 6rem;
		cursor: pointer;
		text-align: center;
		text-decoration: none;

		&:hover {
			color: white;
		}
	}

	.scrolled-nav {
		background-color: black;
		box-shadow:
			0 4px 6px -1px rgba(0, 0, 0, 0.1),
			0 2px 4px -1px rgba(0, 0, 0, 0.6);

		> nav {
			padding: 8px 5%;

			.branding {
				img {
					width: 40px;
					box-shadow:
						0 4px 6px -1px rgba(0, 0, 0, 0.1),
						0 2px 4px -1px rgba(0, 0, 0, 0.6);
				}
			}
		}
	}
}
</style>
