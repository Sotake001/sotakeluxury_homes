<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sotake Luxury Homes - Interactive Land Catalog</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Calm & Balanced -->
    <!-- Application Structure Plan: The application is structured as a single-page interactive dashboard. This design was chosen to transform the static catalog into a dynamic, user-centric tool. The core components are: 1) A control panel with filters for Location, Price Range, and House Type, empowering users to immediately narrow down options based on their needs. 2) A dynamic grid of property cards that visually represents the filtered results, offering a clean, modern, and scannable view. 3) An interactive data visualization section with multiple charts that update in real-time. This provides valuable market insights (e.g., price comparisons, value analysis) that are not apparent from a simple list. The user flow is designed for exploration and discovery, moving from broad filtering to specific property details, making the process of finding the right land investment intuitive and engaging. -->
    <!-- Visualization & Content Choices: 
        - Property Listing: (Source: Catalog Data) -> Goal: Inform/Organize -> Viz: Interactive Cards Grid (HTML/Tailwind/JS) -> Interaction: Dynamically filters based on user controls. -> Justification: Modern, responsive, and allows users to easily scan and select properties of interest.
        - Price Comparison by Location: (Source: Calculated Average Price/SQM) -> Goal: Compare -> Viz: Bar Chart (Chart.js/Canvas) -> Interaction: Updates with filters to show relevant locations. -> Justification: Bar chart is ideal for comparing a quantitative value (price) across different categorical options (locations), highlighting investment value.
        - Property Mix by Type: (Source: Catalog Data) -> Goal: Show Composition -> Viz: Donut Chart (Chart.js/Canvas) -> Interaction: Updates to reflect the composition of the filtered properties. -> Justification: A donut chart effectively shows the proportion of different house types available within the user's selection.
        - Price Range Distribution: (Source: Catalog Data) -> Goal: Organize/Inform -> Viz: Bar Chart (Chart.js/Canvas) -> Interaction: Static chart providing an overview of the entire portfolio's price distribution. -> Justification: Helps users understand the overall market segments Sotake serves, from accessible to premium.
        - Aspirational Property Description: (Source: LLM Generation) -> Goal: Inform/Engage -> Viz: Dynamic Text (HTML/JS) -> Interaction: Button click triggers LLM call. -> Justification: Provides rich, descriptive content to enhance property appeal.
        - Investment Insight: (Source: LLM Generation) -> Goal: Inform/Educate -> Viz: Dynamic Text (HTML/JS) -> Interaction: Button click triggers LLM call based on location. -> Justification: Offers quick, valuable market analysis for decision-making.
    -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #F8F7F4; /* Warm Neutral Background */
        }
        .chart-container {
            position: relative;
            width: 100%;
            height: 300px;
            max-height: 350px;
        }
        @media (min-width: 768px) {
            .chart-container {
                height: 350px;
            }
        }
        /* Custom scrollbar for better aesthetics */
        ::-webkit-scrollbar {
            width: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #F8F7F4;
        }
        ::-webkit-scrollbar-thumb {
            background: #D1C7B9; /* Accent Color */
            border-radius: 10px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #BFAE99;
        }
        .control-panel-bg {
            background-color: #FFFFFF;
        }
        .card-bg {
            background-color: #FFFFFF;
        }
        .header-bg {
            background-color: #EAE6DE; /* Lighter Neutral */
        }
        .accent-bg {
            background-color: #8D877A; /* Muted Accent */
        }
        .accent-text {
            color: #8D877A;
        }
        .primary-text {
            color: #4A453D; /* Dark Neutral Text */
        }
        .secondary-text {
            color: #78716C;
        }
        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 20px;
            height: 20px;
            background: #8D877A;
            cursor: pointer;
            border-radius: 50%;
        }
        input[type="range"]::-moz-range-thumb {
            width: 20px;
            height: 20px;
            background: #8D877A;
            cursor: pointer;
            border-radius: 50%;
        }
        .loading-spinner {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #8D877A;
            border-radius: 50%;
            width: 24px;
            height: 24px;
            animation: spin 1s linear infinite;
            margin: 0 auto;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="primary-text">

    <div class="min-h-screen">
        <!-- Header -->
        <header class="header-bg p-4 md:p-6 shadow-md">
            <div class="container mx-auto">
                <h1 class="text-3xl md:text-4xl font-bold primary-text">Sotake Luxury Homes</h1>
                <p class="mt-1 secondary-text">Interactive Land Investment Catalog - Abuja</p>
            </div>
        </header>

        <!-- Main Content -->
        <main class="container mx-auto p-4 md:p-6">

            <!-- Control Panel -->
            <div id="controls" class="control-panel-bg p-6 rounded-xl shadow-lg mb-8">
                <h2 class="text-2xl font-bold mb-4 border-b pb-2 primary-text">Explore Our Portfolio</h2>
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
                    <div>
                        <label for="locationFilter" class="block text-sm font-medium secondary-text mb-1">Location</label>
                        <select id="locationFilter" class="w-full p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-opacity-50 focus:ring-[#BFAE99] focus:border-[#BFAE99]">
                            <option value="all">All Locations</option>
                        </select>
                    </div>
                    <div>
                        <label for="houseTypeFilter" class="block text-sm font-medium secondary-text mb-1">Proposed House Type</label>
                        <select id="houseTypeFilter" class="w-full p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-opacity-50 focus:ring-[#BFAE99] focus:border-[#BFAE99]">
                            <option value="all">All Types</option>
                        </select>
                    </div>
                    <div class="lg:col-span-2">
                        <label for="priceFilter" class="block text-sm font-medium secondary-text mb-1">Max Price: <span id="priceValue" class="font-bold accent-text"></span></label>
                        <input type="range" id="priceFilter" min="0" max="30000000" step="100000" class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer">
                    </div>
                </div>
                 <div class="text-right mt-4">
                    <button id="resetButton" class="px-4 py-2 accent-bg text-white rounded-lg hover:bg-[#BFAE99] transition-colors">Reset Filters</button>
                </div>
            </div>

            <!-- Property Listings Grid -->
            <section>
                 <div class="flex justify-between items-center mb-4">
                    <h2 class="text-2xl font-bold primary-text">Available Properties</h2>
                    <div id="propertyCount" class="text-lg font-semibold secondary-text"></div>
                </div>
                <div id="property-grid" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
                    <!-- Property cards will be injected here by JS -->
                </div>
                <div id="no-results" class="hidden text-center p-10 card-bg rounded-lg shadow">
                    <p class="text-xl secondary-text">No properties match your current filters.</p>
                </div>
            </section>

            <!-- Data Visualization Dashboard -->
            <section class="mt-12">
                <h2 class="text-3xl font-bold text-center mb-8 primary-text">Portfolio Insights</h2>
                <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                    <div class="card-bg p-6 rounded-xl shadow-lg">
                        <h3 class="text-xl font-bold mb-4 primary-text text-center">Average Price by Location</h3>
                         <p class="text-sm secondary-text text-center mb-4">This chart compares the average price for a 450SQM plot across our various estate locations, giving you an insight into the investment value of each area.</p>
                        <div class="chart-container">
                            <canvas id="locationPriceChart"></canvas>
                        </div>
                    </div>
                    <div class="card-bg p-6 rounded-xl shadow-lg">
                        <h3 class="text-xl font-bold mb-4 primary-text text-center">Property Mix (Filtered)</h3>
                        <p class="text-sm secondary-text text-center mb-4">See the distribution of available proposed house types based on your current filter selection. This helps you understand the variety within your chosen criteria.</p>
                        <div class="chart-container">
                            <canvas id="houseTypeDistributionChart"></canvas>
                        </div>
                    </div>
                     <div class="lg:col-span-2 card-bg p-6 rounded-xl shadow-lg">
                        <h3 class="text-xl font-bold mb-4 primary-text text-center">Portfolio Price Distribution</h3>
                         <p class="text-sm secondary-text text-center mb-4">This chart provides a complete overview of our portfolio, showing how many property options fall into different price brackets.</p>
                        <div class="chart-container" style="height: 400px; max-height:450px;">
                            <canvas id="priceRangeDistributionChart"></canvas>
                        </div>
                    </div>
                </div>
            </section>

            <!-- LLM Powered Insights Section -->
            <section class="mt-12">
                <h2 class="text-3xl font-bold text-center mb-8 primary-text">AI-Powered Insights ✨</h2>
                <div class="card-bg p-6 rounded-xl shadow-lg">
                    <h3 class="text-xl font-bold mb-4 primary-text">Investment Insight by Location</h3>
                    <p class="text-sm secondary-text mb-4">Select a location to get a brief, AI-generated insight into its investment potential.</p>
                    <div class="flex flex-col md:flex-row items-center gap-4">
                        <select id="insightLocationFilter" class="w-full md:w-1/2 p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-opacity-50 focus:ring-[#BFAE99] focus:border-[#BFAE99]">
                            <option value="">Select a Location</option>
                        </select>
                        <button id="getInsightButton" class="px-6 py-2 accent-bg text-white rounded-lg hover:bg-[#BFAE99] transition-colors flex items-center justify-center w-full md:w-auto" disabled>
                            <span id="insightButtonText">Get Insight ✨</span>
                            <div id="insightLoadingSpinner" class="loading-spinner ml-2 hidden"></div>
                        </button>
                    </div>
                    <div id="investmentInsightOutput" class="mt-6 p-4 bg-gray-50 rounded-lg border border-gray-200 secondary-text min-h-[80px] flex items-center justify-center text-center">
                        Select a location and click "Get Insight" to learn more.
                    </div>
                </div>
            </section>

        </main>
    </div>

    <script>
        const propertyData = [
            { location: 'AA3 Kuje', estate: 'AA3 Sapphire City Estate', size: 250, houseType: '3-Bed Terrace', price: 4000000 },
            { location: 'AA3 Kuje', estate: 'AA3 Sapphire City Estate', size: 350, houseType: '3-Bed Bungalow', price: 5500000 },
            { location: 'AA3 Kuje', estate: 'AA3 Sapphire City Estate', size: 450, houseType: '4-Bed Penthouse', price: 7000000 },
            { location: 'AA3 Kuje', estate: 'AA3 Sapphire City Estate', size: 450, houseType: '4-Bed Duplex', price: 7500000 },
            
            { location: 'Pyakasa', estate: 'Bluebells Estate', size: 250, houseType: '3-Bed Terrace', price: 5200000 },
            { location: 'Pyakasa', estate: 'Bluebells Estate', size: 350, houseType: '3-Bed Bungalow', price: 6200000 },
            { location: 'Pyakasa', estate: 'Bluebells Estate', size: 400, houseType: '4-Bed Penthouse', price: 7500000 },
            { location: 'Pyakasa', estate: 'Bluebells Estate', size: 450, houseType: '4-Bed Duplex', price: 9300000 },

            { location: 'Aco Lugbe', estate: 'Aco Estate', size: 250, houseType: '3-Bed Terrace', price: 5000000 },
            { location: 'Aco Lugbe', estate: 'Aco Estate', size: 350, houseType: '3-Bed Bungalow', price: 7000000 },
            { location: 'Aco Lugbe', estate: 'Aco Estate', size: 400, houseType: '4-Bed Penthouse', price: 8000000 },
            { location: 'Aco Lugbe', estate: 'Aco Estate', size: 450, houseType: '4-Bed Duplex', price: 9000000 },

            { location: 'Lifecamp', estate: 'Marble Estate', size: 250, houseType: '3-Bed Terrace', price: 15000000 },
            { location: 'Lifecamp', estate: 'Marble Estate', size: 350, houseType: '3-Bed Bungalow', price: 22000000 },
            { location: 'Lifecamp', estate: 'Marble Estate', size: 400, houseType: '4-Bed Penthouse', price: 23000000 },
            { location: 'Lifecamp', estate: 'Marble Estate', size: 450, houseType: '4-Bed Duplex', price: 25000000 },

            { location: 'Karsana', estate: 'Cedar Crest Estate', size: 250, houseType: '3-Bed Terrace', price: 15000000 },
            { location: 'Karsana', estate: 'Cedar Crest Estate', size: 350, houseType: '3-Bed Bungalow', price: 25000000 },
            { location: 'Karsana', estate: 'Cedar Crest Estate', size: 400, houseType: '4-Bed Penthouse', price: 28000000 },
            { location: 'Karsana', estate: 'Cedar Crest Estate', size: 450, houseType: '4-Bed Duplex', price: 30000000 },

            { location: 'Guzape', estate: 'Regal Estate', size: 250, houseType: '3-Bed Terrace', price: 15000000 },
            { location: 'Guzape', estate: 'Regal Estate', size: 350, houseType: '3-Bed Bungalow', price: 22000000 },
            { location: 'Guzape', estate: 'Regal Estate', size: 400, houseType: '4-Bed Penthouse', price: 26000000 },
            { location: 'Guzape', estate: 'Regal Estate', size: 450, houseType: '4-Bed Duplex', price: 28000000 },

            { location: 'Kuchako', estate: 'Diamond City', size: 250, houseType: '3-Bed Terrace', price: 2000000 },
            { location: 'Kuchako', estate: 'Diamond City', size: 350, houseType: '3-Bed Bungalow', price: 2500000 },
            { location: 'Kuchako', estate: 'Diamond City', size: 450, houseType: '4-Bed Penthouse', price: 3000000 },
            { location: 'Kuchako', estate: 'Diamond City', size: 450, houseType: '4-Bed Duplex', price: 4000000 },

            { location: 'Gwagwalada', estate: 'Kutunku', size: 250, houseType: '3-Bed Terrace', price: 2000000 },
            { location: 'Gwagwalada', estate: 'Kutunku', size: 350, houseType: '3-Bed Bungalow', price: 1400000 },
            { location: 'Gwagwalada', estate: 'Kutunku', size: 400, houseType: '4-Bed Penthouse', price: 2500000 },
            { location: 'Gwagwalada', estate: 'Kutunku', size: 450, houseType: '4-Bed Duplex', price: 2300000 },

            { location: 'Idu', estate: 'Idu City', size: 250, houseType: '3-Bed Terrace', price: 4500000 },
            { location: 'Idu', estate: 'Idu City', size: 350, houseType: '3-Bed Bungalow', price: 6500000 },
            { location: 'Idu', estate: 'Idu City', size: 400, houseType: '4-Bed Penthouse', price: 7500000 },
            { location: 'Idu', estate: 'Idu City', size: 450, houseType: '4-Bed Duplex', price: 8500000 },
        ];

        const locationFilter = document.getElementById('locationFilter');
        const houseTypeFilter = document.getElementById('houseTypeFilter');
        const priceFilter = document.getElementById('priceFilter');
        const priceValue = document.getElementById('priceValue');
        const resetButton = document.getElementById('resetButton');
        const propertyGrid = document.getElementById('property-grid');
        const noResults = document.getElementById('no-results');
        const propertyCount = document.getElementById('propertyCount');
        const insightLocationFilter = document.getElementById('insightLocationFilter');
        const getInsightButton = document.getElementById('getInsightButton');
        const insightButtonText = document.getElementById('insightButtonText');
        const insightLoadingSpinner = document.getElementById('insightLoadingSpinner');
        const investmentInsightOutput = document.getElementById('investmentInsightOutput');

        let charts = {};

        const formatPrice = (price) => `₦${(price / 1000000).toFixed(1)}M`;
        const formatFullPrice = (price) => `₦${price.toLocaleString()}`;

        function populateFilters() {
            const locations = [...new Set(propertyData.map(p => p.location))];
            locations.sort().forEach(location => {
                const option = document.createElement('option');
                option.value = location;
                option.textContent = location;
                locationFilter.appendChild(option);

                const insightOption = document.createElement('option');
                insightOption.value = location;
                insightOption.textContent = location;
                insightLocationFilter.appendChild(insightOption);
            });

            const houseTypes = [...new Set(propertyData.map(p => p.houseType))];
            houseTypes.sort().forEach(type => {
                const option = document.createElement('option');
                option.value = type;
                option.textContent = type;
                houseTypeFilter.appendChild(option);
            });
            
            const maxPrice = Math.max(...propertyData.map(p => p.price));
            priceFilter.max = maxPrice;
            priceFilter.value = maxPrice;
            priceValue.textContent = formatPrice(maxPrice);
        }

        function renderProperties(properties) {
            propertyGrid.innerHTML = '';
            propertyCount.textContent = `${properties.length} options found`;

            if (properties.length === 0) {
                noResults.classList.remove('hidden');
                propertyGrid.classList.add('hidden');
                return;
            }
            noResults.classList.add('hidden');
            propertyGrid.classList.remove('hidden');

            properties.forEach((p, index) => {
                const card = document.createElement('div');
                card.className = 'card-bg rounded-xl shadow-lg overflow-hidden transform hover:-translate-y-1 transition-transform duration-300';
                card.innerHTML = `
                    <div class="p-5">
                        <p class="text-sm font-semibold accent-text uppercase tracking-wider">${p.estate}</p>
                        <h3 class="text-xl font-bold primary-text mt-1">${p.location}</h3>
                        <div class="mt-4 space-y-2">
                            <p class="secondary-text"><span class="font-semibold primary-text">Plot Size:</span> ${p.size}SQM</p>
                            <p class="secondary-text"><span class="font-semibold primary-text">House Type:</span> ${p.houseType}</p>
                        </div>
                        <div class="mt-4 pt-4 border-t border-gray-200">
                             <p class="text-2xl font-bold primary-text">${formatPrice(p.price)}</p>
                             <p class="text-xs secondary-text">${formatFullPrice(p.price)}</p>
                        </div>
                        <div class="mt-4">
                            <button class="generate-description-btn px-4 py-2 bg-gray-200 text-primary-text rounded-lg hover:bg-gray-300 transition-colors text-sm w-full" data-index="${index}">
                                Generate Aspirational Description ✨
                            </button>
                            <div class="generated-description mt-2 text-sm secondary-text bg-gray-50 p-3 rounded hidden"></div>
                            <div class="description-loading-spinner mt-2 hidden loading-spinner"></div>
                        </div>
                    </div>
                    <div class="accent-bg p-3 text-center text-white font-bold cursor-pointer hover:bg-[#BFAE99] transition-colors">
                        Inquire Now
                    </div>
                `;
                propertyGrid.appendChild(card);
            });

            document.querySelectorAll('.generate-description-btn').forEach(button => {
                button.addEventListener('click', async (event) => {
                    const index = event.target.dataset.index;
                    const prop = properties[index];
                    const outputDiv = event.target.nextElementSibling;
                    const spinner = outputDiv.nextElementSibling;

                    outputDiv.classList.add('hidden');
                    spinner.classList.remove('hidden');
                    event.target.disabled = true;

                    const prompt = `Generate a concise, aspirational, and luxurious marketing description (around 60-80 words) for a land plot in ${prop.estate}, ${prop.location}. The plot is ${prop.size}SQM and is ideal for building a ${prop.houseType}. Focus on the lifestyle, investment potential, and exclusivity.`;
                    
                    try {
                        const description = await callGeminiAPI(prompt);
                        outputDiv.textContent = description;
                        outputDiv.classList.remove('hidden');
                    } catch (error) {
                        outputDiv.textContent = 'Failed to generate description. Please try again.';
                        outputDiv.classList.remove('hidden');
                        console.error('Gemini API error:', error);
                    } finally {
                        spinner.classList.add('hidden');
                        event.target.disabled = false;
                    }
                });
            });
        }
        
        const chartTooltipOptions = {
            plugins: {
                tooltip: {
                    callbacks: {
                        title: function(tooltipItems) {
                            const item = tooltipItems[0];
                            let label = item.chart.data.labels[item.dataIndex];
                            if (Array.isArray(label)) {
                              return label.join(' ');
                            }
                            return label;
                        },
                         label: function(context) {
                            let label = context.dataset.label || '';
                            if (label) {
                                label += ': ';
                            }
                            if (context.parsed.y !== null) {
                                label += `₦${(context.parsed.y / 1000000)}M`;
                            }
                            return label;
                        }
                    }
                }
            }
        };
        
        const chartOptions = {
             maintainAspectRatio: false,
             responsive: true,
             ...chartTooltipOptions
        }

        function renderLocationPriceChart() {
            const ctx = document.getElementById('locationPriceChart').getContext('2d');
            const locations = [...new Set(propertyData.map(p => p.location))];
            
            const data = locations.map(loc => {
                const props = propertyData.filter(p => p.location === loc && p.size === 450);
                const avgPrice = props.length > 0 ? props.reduce((acc, curr) => acc + curr.price, 0) / props.length : 0;
                return { location: loc, price: avgPrice };
            }).sort((a, b) => a.price - b.price);


            if (charts.locationPrice) charts.locationPrice.destroy();
            charts.locationPrice = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: data.map(d => d.location),
                    datasets: [{
                        label: 'Avg. Price for 450SQM Plot',
                        data: data.map(d => d.price),
                        backgroundColor: '#8D877A',
                        borderColor: '#78716C',
                        borderWidth: 1
                    }]
                },
                options: {
                    ...chartOptions,
                    scales: {
                        y: {
                            beginAtZero: true,
                             ticks: {
                                callback: function(value, index, values) {
                                    return `₦${(value / 1000000)}M`;
                                }
                            }
                        }
                    }
                }
            });
        }

        function renderHouseTypeDistributionChart(properties) {
            const ctx = document.getElementById('houseTypeDistributionChart').getContext('2d');
            const typeCounts = properties.reduce((acc, p) => {
                acc[p.houseType] = (acc[p.houseType] || 0) + 1;
                return acc;
            }, {});

            const labels = Object.keys(typeCounts);
            const data = Object.values(typeCounts);

            if (charts.houseType) charts.houseType.destroy();
            charts.houseType = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: labels,
                    datasets: [{
                        label: 'Property Types',
                        data: data,
                        backgroundColor: ['#8D877A', '#BFAE99', '#D1C7B9', '#EAE6DE'],
                        borderColor: '#FFFFFF',
                        borderWidth: 2
                    }]
                },
                options: { ...chartOptions, plugins: {
                    ...chartTooltipOptions.plugins,
                     legend: { position: 'bottom' }
                } }
            });
        }

        function renderPriceRangeDistributionChart() {
            const ctx = document.getElementById('priceRangeDistributionChart').getContext('2d');
            const priceRanges = {
                'Under ₦5M': propertyData.filter(p => p.price < 5000000).length,
                '₦5M - ₦10M': propertyData.filter(p => p.price >= 5000000 && p.price < 10000000).length,
                '₦10M - ₦20M': propertyData.filter(p => p.price >= 10000000 && p.price < 20000000).length,
                '₦20M - ₦30M': propertyData.filter(p => p.price >= 20000000 && p.price <= 30000000).length
            };

            if(charts.priceRange) charts.priceRange.destroy();
            charts.priceRange = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: Object.keys(priceRanges),
                    datasets: [{
                        label: 'Number of Property Options',
                        data: Object.values(priceRanges),
                        backgroundColor: ['#8D877A', '#BFAE99', '#D1C7B9', '#EAE6DE'],
                    }]
                },
                options: {
                    ...chartOptions,
                     indexAxis: 'y',
                    scales: {
                        x: {
                            beginAtZero: true,
                            ticks: { stepSize: 1 }
                        }
                    },
                    plugins: {
                        ...chartTooltipOptions.plugins,
                        tooltip: {
                            callbacks: {
                                label: context => ` ${context.raw} properties`
                            }
                        }
                    }
                }
            });
        }

        async function callGeminiAPI(prompt) {
            let chatHistory = [];
            chatHistory.push({ role: "user", parts: [{ text: prompt }] });
            const payload = { contents: chatHistory };
            const apiKey = ""; 
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;

            const response = await fetch(apiUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });
            const result = await response.json();

            if (result.candidates && result.candidates.length > 0 &&
                result.candidates[0].content && result.candidates[0].content.parts &&
                result.candidates[0].content.parts.length > 0) {
                return result.candidates[0].content.parts[0].text;
            } else {
                throw new Error('Invalid API response structure or no content.');
            }
        }

        function filterAndRender() {
            const selectedLocation = locationFilter.value;
            const selectedHouseType = houseTypeFilter.value;
            const maxPrice = priceFilter.value;

            priceValue.textContent = formatPrice(maxPrice);

            const filteredProperties = propertyData.filter(p => {
                const locationMatch = selectedLocation === 'all' || p.location === selectedLocation;
                const typeMatch = selectedHouseType === 'all' || p.houseType === selectedHouseType;
                const priceMatch = p.price <= maxPrice;
                return locationMatch && typeMatch && priceMatch;
            });

            renderProperties(filteredProperties);
            renderHouseTypeDistributionChart(filteredProperties);
        }
        
        function resetAll() {
            locationFilter.value = 'all';
            houseTypeFilter.value = 'all';
            priceFilter.value = Math.max(...propertyData.map(p => p.price));
            insightLocationFilter.value = '';
            investmentInsightOutput.textContent = 'Select a location and click "Get Insight" to learn more.';
            getInsightButton.disabled = true;
            filterAndRender();
        }

        locationFilter.addEventListener('change', filterAndRender);
        houseTypeFilter.addEventListener('change', filterAndRender);
        priceFilter.addEventListener('input', filterAndRender);
        resetButton.addEventListener('click', resetAll);

        insightLocationFilter.addEventListener('change', () => {
            getInsightButton.disabled = insightLocationFilter.value === '';
            investmentInsightOutput.textContent = 'Select a location and click "Get Insight" to learn more.';
        });

        getInsightButton.addEventListener('click', async () => {
            const selectedLocation = insightLocationFilter.value;
            if (!selectedLocation) {
                investmentInsightOutput.textContent = 'Please select a location first.';
                return;
            }

            insightButtonText.classList.add('hidden');
            insightLoadingSpinner.classList.remove('hidden');
            getInsightButton.disabled = true;
            investmentInsightOutput.textContent = 'Generating insight...';

            const prompt = `Provide a brief (around 50-70 words) investment insight for real estate in ${selectedLocation}, Abuja. Highlight its key appeal for property investors, considering factors like growth potential, infrastructure, or unique market position.`;

            try {
                const insight = await callGeminiAPI(prompt);
                investmentInsightOutput.textContent = insight;
            } catch (error) {
                investmentInsightOutput.textContent = 'Failed to generate insight. Please try again.';
                console.error('Gemini API error:', error);
            } finally {
                insightButtonText.classList.remove('hidden');
                insightLoadingSpinner.classList.add('hidden');
                getInsightButton.disabled = false;
            }
        });

        document.addEventListener('DOMContentLoaded', () => {
            populateFilters();
            renderProperties(propertyData);
            renderLocationPriceChart();
            renderHouseTypeDistributionChart(propertyData);
            renderPriceRangeDistributionChart();
        });
    </script>
    </body>
</html>
