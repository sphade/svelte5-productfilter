<script lang="ts">
	import Product from '$lib/components/Product.svelte';
	import * as DropdownMenu from '$lib/components/ui/dropdown-menu';
	import ChevronDown from 'lucide-svelte/icons/chevron-down';
	import Filter from 'lucide-svelte/icons/filter';
	import * as Accordion from '$lib/components/ui/accordion';
	import { Slider } from '$lib/components/ui/slider';
	import { unstate, untrack } from 'svelte';
	import type { ProductState } from '$lib/validators/product-validator';
	import { cn } from '$lib/utils.js';
	import { enhance } from '$app/forms';
	import type { Product as TProduct } from '$lib/server/db';
	import type { QueryResult } from '@upstash/vector';
	import ProductSkeleton from '$lib/components/ProductSkeleton.svelte';
	import debounce from 'lodash.debounce';
	const SORT_OPTIONS = [
		{ name: 'none', value: 'none' },
		{ name: 'Price : Low to High', value: 'price-asc' },
		{ name: 'Price : High to Low  ', value: 'price-desc' }
	] as const;

	const SUBCATEGORIES = [
		{ name: 'T-Shirts', selected: true, href: '#' },
		{ name: 'Hoodies', selected: false, href: '#' },
		{ name: 'Sweatshirts', selected: false, href: '#' },
		{ name: 'Accessories', selected: false, href: '#' }
	];

	const COLOR_FILTERS = {
		id: 'color',
		name: 'Color',
		options: [
			{ value: 'white', label: 'White' },
			{ value: 'beige', label: 'Beige' },
			{ value: 'blue', label: 'Blue' },
			{ value: 'green', label: 'Green' },
			{ value: 'purple', label: 'Purple' }
		] as const
	};
	const SIZE_FILTERS = {
		id: 'size',
		name: 'Size',
		options: [
			{ value: 'S', label: 'S' },
			{ value: 'M', label: 'M' },
			{ value: 'L', label: 'L' }
		]
	} as const;
	const PRICE_FILTERS = {
		id: 'price',
		name: 'Price',
		options: [
			{ value: [0, 100], label: 'any price' },
			{ value: [0, 20], label: 'under $20' },
			{ value: [0, 40], label: 'under $40' }
		]
	} as const;

	const DEFAULT_CUSTOM_PRICE = [0, 100] as [number, number];
	let filter = $state<ProductState>({
		color: ['beige', 'blue', 'green', 'purple', 'white'],
		price: { isCustom: false, range: DEFAULT_CUSTOM_PRICE },
		size: ['L', 'M', 'S'],
		sort: 'none'
	});
	function applyArrayFilter({
		category,
		value
	}: {
		category: keyof Omit<typeof filter, 'price' | 'sort'>;
		value: string;
	}) {
		const isFilterApplied = filter[category].includes(value as never);
		if (isFilterApplied) {
			filter[category] = filter[category].filter((v) => v !== value) as typeof filter.color &
				typeof filter.size;
		} else {
			filter[category].push(value as never);
		}
	}
	const minPrice = $derived(Math.min(filter.price.range[0], filter.price.range[1]));
	const maxPrice = $derived(Math.max(filter.price.range[0], filter.price.range[1]));
	async function getProducts() {
		const products = (await (
			await fetch('http://localhost:5173/api/products', {
				method: 'POST',
				body: JSON.stringify({
					filter: {
						sort: filter.sort,
						color: filter.color,
						price: filter.price.range,
						size: filter.size
					}
				}),
				headers: {
					'content-type': 'application/json'
				}
			})
		).json()) as QueryResult<TProduct>[];

		return products;
	}

	const onSubmit = (range: number[]) => {
		const [newMin, newMax] = range;
		filter.price = {
			isCustom: true,
			range: [newMin, newMax]
		};
	};

	const debouncedSubmit = debounce(onSubmit, 400);
</script>

<main class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
	<div class="flex items-baseline justify-between border-b border-gray-200 pb-6 pt-24">
		<h1 class="text-4xl font-bold capitalize tracking-tight text-gray-900">
			high quality cotton selection
		</h1>
		<div class="flex items-center">
			<DropdownMenu.Root>
				<DropdownMenu.Trigger
					class="group inline-flex justify-center text-sm font-medium text-gray-700 hover:text-gray-900"
					>sort
					<ChevronDown />
				</DropdownMenu.Trigger>
				<DropdownMenu.Content align="end">
					{#each SORT_OPTIONS as { name, value }}
						<button
							onclick={() => {
								filter.sort = value;
							}}
							class="block w-full p-2 px-4 text-left text-sm"
							class:text-gray-900={value === filter.sort}
							class:bg-gray-100={value === filter.sort}
							class:text-gray-500={value === filter.sort}>{name}</button
						>
					{/each}
				</DropdownMenu.Content>
			</DropdownMenu.Root>
			<button class="-m-2 ml-4 text-gray-400 hover:text-gray-500 sm:ml-6 lg:hidden">
				<Filter class="size-5" />
			</button>
		</div>
	</div>
	<section class="pb-24 pt-6">
		<div class="grid grid-cols-1 gap-x-8 gap-y-10 lg:grid-cols-4">
			<!-- filter -->
			<div class="hidden lg:block">
				<ul class="space-y-4 border-b border-gray-200 pb-6 text-sm font-medium text-gray-500">
					{#each SUBCATEGORIES as { href, name, selected }}
						<li>
							<button disabled={!selected} class="disabled:cursor-not-allowed disabled:opacity-60"
								>{name}
							</button>
						</li>
					{/each}
				</ul>
				<Accordion.Root multiple class="">
					<Accordion.Item value="color">
						<Accordion.Trigger class="py-3 text-sm text-gray-400 hover:text-gray-500">
							<span class="font-medium text-gray-900"> color </span>
						</Accordion.Trigger>
						<Accordion.Content class="pt-6 ">
							<ul class="space-y-4">
								{#each COLOR_FILTERS.options as { label, value }, id}
									<li class="flex items-center">
										<input
											type="checkbox"
											id={`color-${id}`}
											class="size-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500"
											onchange={() =>
												applyArrayFilter({
													category: 'color',
													value
												})}
											checked={filter.color.includes(value)}
										/>
										<label for={`color-${id}`} class="ml-3 text-sm text-gray-600">
											{label}
										</label>
									</li>
								{/each}
							</ul>
						</Accordion.Content>
					</Accordion.Item>
					<Accordion.Item value="size">
						<Accordion.Trigger class="py-3 text-sm text-gray-400 hover:text-gray-500">
							<span class="font-medium text-gray-900"> size </span>
						</Accordion.Trigger>
						<Accordion.Content class="pt-6 ">
							<ul class="space-y-4">
								{#each SIZE_FILTERS.options as { label, value }, id}
									<li class="flex items-center">
										<input
											type="checkbox"
											id={`size-${id}`}
											class="size-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500"
											onchange={() =>
												applyArrayFilter({
													category: 'size',
													value
												})}
											checked={filter.size.includes(value)}
										/>
										<label for={`size-${id}`} class="ml-3 text-sm text-gray-600">
											{label}
										</label>
									</li>
								{/each}
							</ul>
						</Accordion.Content>
					</Accordion.Item>
					<Accordion.Item value="price">
						<Accordion.Trigger class="py-3 text-sm text-gray-400 hover:text-gray-500">
							<span class="font-medium text-gray-900"> price </span>
						</Accordion.Trigger>
						<Accordion.Content class="pt-6 ">
							<ul class="space-y-4">
								{#each PRICE_FILTERS.options as { label, value }, id}
									<li class="flex items-center">
										<input
											type="radio"
											id={`price-${id}`}
											class="size-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500"
											onchange={() => {
												filter.price = {
													isCustom: false,
													range: [...value]
												};
											}}
											checked={!filter.price.isCustom &&
												filter.price.range[0] === value[0] &&
												filter.price.range[1] === value[1]}
										/>
										<label for={`price-${id}`} class="ml-3 text-sm text-gray-600">
											{label}
										</label>
									</li>
								{/each}
								<li class="flex flex-col justify-center gap-2">
									<div>
										<input
											type="radio"
											id="price-{PRICE_FILTERS.options.length}"
											class="size-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500"
											onchange={() => {
												filter.price = {
													isCustom: true,
													range: [0, 100]
												};
											}}
											checked={filter.price.isCustom}
										/>
										<label
											for={`price-${PRICE_FILTERS.options.length}`}
											class="ml-3 text-sm text-gray-600"
										>
											Custom
										</label>
									</div>
									<div class="flex justify-between">
										<p class="font-medium">Price</p>
										<div>
											{filter.price.isCustom
												? minPrice.toFixed(0)
												: filter.price.range[0].toFixed(0)}{' '}
											$ -{' '}
											{filter.price.isCustom
												? maxPrice.toFixed(0)
												: filter.price.range[1].toFixed(0)}{' '}
											$
										</div>
									</div>
									<div class="px-3">
										<Slider
											value={filter.price.isCustom ? filter.price.range : DEFAULT_CUSTOM_PRICE}
											max={DEFAULT_CUSTOM_PRICE[1]}
											min={DEFAULT_CUSTOM_PRICE[0]}
											step={5}
											class={cn(!filter.price.isCustom && 'opacity-50')}
											disabled={!filter.price.isCustom}
											onValueChange={debouncedSubmit}
										/>
									</div>
								</li>
							</ul>
						</Accordion.Content>
					</Accordion.Item>
				</Accordion.Root>
			</div>
			<ul class="grid grid-cols-1 gap-8 sm:grid-cols-2 md:grid-cols-3 lg:col-span-3">
				{#await getProducts()}
					{#each { length: 12 } as _}
						<ProductSkeleton />
					{/each}
				{:then products}
					{#each products as product}
						<Product product={product.metadata!} />
					{/each}
				{/await}
			</ul>
		</div>
	</section>
</main>
