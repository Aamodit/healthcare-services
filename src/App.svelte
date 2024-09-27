<script>
  import { writable } from 'svelte/store';
  import { fade, slide } from 'svelte/transition';

  /** @type {{ id: number, name: string, description: string, price: number }[]} */
  let services = writable([
    { id: 1, name: 'General Checkup', description: 'Annual physical examination', price: 100 },
    { id: 2, name: 'Dental Cleaning', description: 'Professional teeth cleaning', price: 75 },
    { id: 3, name: 'Blood Test', description: 'Comprehensive blood analysis', price: 50 },
    { id: 4, name: 'X-Ray', description: 'Radiographic imaging for diagnosis', price: 120 },
    { id: 5, name: 'Vaccination', description: 'Preventative immunization shots', price: 30 },
  ]);

  let nextId = 3;
  let name = '';
  let description = '';
  let price = '';
  let editingId = null;
  let errorMessage = '';

  function validateForm() {
    if (!name.trim()) {
      errorMessage = 'Service name is required';
      return false;
    }
    if (!description.trim()) {
      errorMessage = 'Description is required';
      return false;
    }
    if (!price || isNaN(parseFloat(price)) || parseFloat(price) <= 0) {
      errorMessage = 'Price must be a positive number';
      return false;
    }
    errorMessage = '';
    return true;
  }

  function addService() {
    if (validateForm()) {
      $services = [...$services, { id: nextId++, name, description, price: parseFloat(price) }];
      resetForm();
    }
  }

  function updateService() {
    if (validateForm() && editingId !== null) {
      $services = $services.map(service => 
        service.id === editingId 
          ? { ...service, name, description, price: parseFloat(price) }
          : service
      );
      resetForm();
    }
  }

  function editService(service) {
    editingId = service.id;
    name = service.name;
    description = service.description;
    price = service.price.toString();
  }

  function deleteService(id) {
    $services = $services.filter(service => service.id !== id);
  }

  function resetForm() {
    name = '';
    description = '';
    price = '';
    editingId = null;
    errorMessage = '';
  }
</script>

<div class="min-h-screen bg-gray-100 py-6 flex flex-col justify-center sm:py-12">
  <div class="relative py-3 sm:max-w-xl sm:mx-auto">
    <div class="absolute inset-0 bg-gradient-to-r from-cyan-400 to-blue-500 shadow-lg transform -skew-y-6 sm:skew-y-0 sm:-rotate-6 sm:rounded-3xl"></div>
    <div class="relative px-4 py-10 bg-white shadow-lg sm:rounded-3xl sm:p-20">
      <div class="max-w-md mx-auto">
        <h1 class="text-3xl font-extrabold text-gray-900 mb-6">Healthcare Services</h1>

        <form on:submit|preventDefault={editingId ? updateService : addService} class="mb-8 space-y-4">
          <div>
            <label for="name" class="block text-sm font-medium text-gray-700">Service Name</label>
            <input type="text" id="name" bind:value={name} required class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50">
          </div>
          <div>
            <label for="description" class="block text-sm font-medium text-gray-700">Description</label>
            <textarea id="description" bind:value={description} required class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"></textarea>
          </div>
          <div>
            <label for="price" class="block text-sm font-medium text-gray-700">Price</label>
            <div class="mt-1 relative rounded-md shadow-sm">
              <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                <span class="text-gray-500 sm:text-sm">$</span>
              </div>
              <input type="number" id="price" bind:value={price} required min="0" step="0.01" class="block w-full pl-7 pr-12 rounded-md border-gray-300 focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm">
            </div>
          </div>
          {#if errorMessage}
            <p class="text-red-500 text-sm" transition:fade>{errorMessage}</p>
          {/if}
          <div class="flex items-center justify-between">
            <button type="submit" class="inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500">
              {editingId ? 'Update Service' : 'Add Service'}
            </button>
            {#if editingId}
              <button type="button" on:click={resetForm} class="inline-flex items-center px-4 py-2 border border-gray-300 shadow-sm text-sm font-medium rounded-md text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500">
                Cancel
              </button>
            {/if}
          </div>
        </form>

        <ul class="space-y-4">
          {#each $services as service (service.id)}
            <li transition:slide|local class="bg-white overflow-hidden shadow rounded-lg">
              <div class="px-4 py-5 sm:p-6">
                <h3 class="text-lg leading-6 font-medium text-gray-900">{service.name}</h3>
                <p class="mt-1 max-w-2xl text-sm text-gray-500">{service.description}</p>
                <p class="mt-1 max-w-2xl text-sm font-bold text-gray-700">${service.price.toFixed(2)}</p>
                <div class="mt-4 flex space-x-3">
                  <button on:click={() => editService(service)} class="inline-flex items-center px-3 py-1.5 border border-transparent text-xs font-medium rounded-full shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500">
                    Edit
                  </button>
                  <button on:click={() => deleteService(service.id)} class="inline-flex items-center px-3 py-1.5 border border-transparent text-xs font-medium rounded-full shadow-sm text-white bg-red-600 hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-red-500">
                    Delete
                  </button>
                </div>
              </div>
            </li>
          {:else}
            <p class="text-center text-gray-500">No services available. Add a new service to get started.</p>
          {/each}
        </ul>
      </div>
    </div>
  </div>
</div>