<script lang="ts">
	import type { Product } from '$lib/server/db';
	let { product }: { product: Product } = $props();
</script>

<div class="group relative">
	<div
		class="aspect-h-1 aspect-w-1 lg:aspect-none w-full overflow-hidden rounded-md bg-gray-200 group-hover:opacity-75 lg:h-80"
	>
		<img src={product.imageId} alt="product" class="h-full w-full object-cover object-center" />
	</div>
	<div class="mt-4 flex justify-between">
		<div>
			<h3 class="text-sm text-gray-700">{product.name}</h3>
			<p class="mt-1 text-sm text-gray-500">
				Size {product.size.toUpperCase()}, {product.color}
			</p>
		</div>

		<p class="text-sm font-medium text-gray-900">{product.price}</p>
	</div>
</div>
