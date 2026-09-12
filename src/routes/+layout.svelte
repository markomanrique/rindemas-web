<script lang="ts">
	import '../app.css';
	import Footer from '$lib/components/Footer.svelte';
	import Header from '$lib/components/Header.svelte';
	import { Toaster, toast } from 'svelte-sonner';
	import { onMount, setContext } from 'svelte';
	import { Loader, X } from 'lucide-svelte';

	let { children } = $props();

	const SHEET_URL =
		'https://script.google.com/macros/s/AKfycbyQsUXOKesuDhaIKtlY6LXjdlHh3W7rx7RKamoaA2cH93js3UtdFCG9I8HDozw3LmDt/exec';

	let dialog = $state<HTMLDialogElement | null>(null);
	let name = $state('');
	let phone = $state('');
	let mail = $state('');
	let licencia = $state<'sí' | 'no' | null>(null);
	let loading = $state(false);
	let error_msg = $state('');

	function openDialog() {
		if (dialog) {
			dialog.showModal();
		}
	}

	setContext('openDialog', openDialog);

	function closeDialog() {
		if (dialog) {
			dialog.close();
		}
	}

	async function onsubmit(e: Event) {
		e.preventDefault();

		loading = true;
		error_msg = '';

		if (!name.trim() || !phone.trim() || !mail.trim() || !licencia) {
			error_msg = 'Por favor completa todos los campos.';
			loading = false;
			return;
		}

		if (!/^\d{9,}$/.test(phone)) {
			error_msg = 'El teléfono debe tener al menos 9 dígitos.';
			loading = false;
			return;
		}

		if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(mail)) {
			error_msg = 'Correo no válido.';
			loading = false;
			return;
		}

		const payload = { name, phone, mail, licencia };

		try {
			await fetch(SHEET_URL, {
				method: 'POST',
				mode: 'no-cors',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify(payload)
			});

			toast.success('Datos enviados correctamente');

			// Limpia el form
			name = phone = mail = '';
			licencia = null;
			loading = false;
			closeDialog();
		} catch (err) {
			console.error('Error al enviar:', err);
			error_msg = 'Hubo un problema al enviar :(';
			loading = false;
		}
	}

	// Popup automático al entrar. En pausa por ahora; poner en true para reactivarlo.
	const POPUP_AL_ENTRAR = false;

	onMount(() => {
		if (POPUP_AL_ENTRAR) openDialog();
	});
</script>

<Toaster richColors />
<Header {openDialog} />
{@render children?.()}
<Footer {openDialog} />

<dialog
	bind:this={dialog}
	onclick={(e) => {
		if (e.target === dialog) {
			closeDialog();
		}
	}}
>
	<form class="flex flex-col gap-5 px-6 py-8" {onsubmit}>
		<h1 class="text-center text-4xl leading-none">¡Únete ahora!</h1>
		<p class="text-center sm:text-lg">
			Déjanos tus datos y nos pondremos en contacto contigo para la entrevista.
		</p>
		<label for="name">
			<span class="font-bold">Nombre</span>
			<input
				type="text"
				id="name"
				name="name"
				placeholder="Ej. Paulo Torres"
				bind:value={name}
				required
			/>
		</label>
		<label for="tel">
			<span class="font-bold">Teléfono</span>
			<input
				type="tel"
				id="tel"
				name="tel"
				placeholder="Ej. 940 185 837"
				inputmode="numeric"
				pattern="[0-9]*"
				bind:value={phone}
				oninput={(e) => (phone = e.currentTarget.value.replace(/\D/g, ''))}
				required
			/>
		</label>
		<label for="mail">
			<span class="font-bold">Correo</span>
			<input
				type="mail"
				id="mail"
				name="mail"
				placeholder="Ej. paulotorres@gmail.com"
				required
				bind:value={mail}
			/>
		</label>
		<label for="licencia">
			<span class="font-bold">¿Tienes Moto?</span>
			<div class="grid grid-cols-2 gap-2 text-center font-bold">
				<button
					type="button"
					class={`block h-12 cursor-pointer content-center rounded-lg outline-blue-400 transition-all focus:outline-3
				${licencia === 'sí' ? 'bg-blue-600 text-white' : 'bg-neutral-700 text-neutral-300 hover:bg-neutral-600'}`}
					onclick={() => {
						licencia = 'sí';
					}}
				>
					<input type="radio" name="licencia" value="sí" bind:group={licencia} class="hidden" />
					Sí
				</button>
				<button
					type="button"
					class={`block h-12 cursor-pointer content-center rounded-lg outline-blue-400 transition-all focus:outline-3
				${licencia === 'no' ? 'bg-blue-600 text-white' : 'bg-neutral-700 text-neutral-300 hover:bg-neutral-600'}`}
					onclick={() => {
						licencia = 'no';
					}}
				>
					<input type="radio" name="licencia" value="no" bind:group={licencia} class="hidden" />
					No
				</button>
			</div>
		</label>
		{#if error_msg}
			<p class="text-red-400"><strong>Error:</strong> {error_msg}</p>
		{/if}
		<button
			type="submit"
			class="btn flex items-center justify-center gap-2 rounded-lg"
			disabled={loading}
		>
			{#if loading}
				<Loader class="size-4 animate-spin" />
			{/if}
			{loading ? 'Enviando' : 'Enviar'}
		</button>
		<button
			type="button"
			class="hover:underline"
			aria-label="Cerrar"
			title="Cerrar"
			onclick={closeDialog}
		>
			Más información
		</button>
	</form>
</dialog>

<style>
	dialog {
		border-radius: 24px;
		border: 2px solid var(--text);
		width: calc(100% - 48px);
		max-width: 540px;
		position: fixed;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		background-color: var(--color-dark-gray);
		color: var(--color-cream);
	}
	dialog:focus {
		outline: none;
	}
	::backdrop {
		background: rgba(0, 0, 0, 0.75);
		backdrop-filter: blur(8px);
	}
</style>
