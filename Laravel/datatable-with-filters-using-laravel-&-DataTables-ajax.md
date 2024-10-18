# Product Data Table with Filters using Laravel & DataTables

This project displays product data in a table format with filtering options using Laravel and DataTables. The interface allows filtering products by customer, supplier, brand, and status. The products data is fetched dynamically through Ajax and rendered in the table.

## Code Explanation

### 1. Frontend: Display Product Table & Filters

The following code represents the frontend structure to display the product table and filtering options.

```html
<div class="card">
    <div class="card-header d-block">
        <div class="row align-items-center mt-5">
            <div class="col-lg-3">
                <div class="d-flex w-100 align-items-center position-relative">
                    <i class="ki-duotone ki-magnifier fs-3 position-absolute ms-4">
                        <span class="path1"></span>
                        <span class="path2"></span>
                    </i>
                    <input type="text" data-kt-ecommerce-order-filter="search" name="customer"
                        id="search_customer" class="form-control form-control-sm w-md-250px w-100 ps-12"
                        placeholder="Search Product">
                </div>
            </div>
            <div class="col-lg-2">
                <select id="filter_supplier" class="form-control w-100 form-select-sm form-select"
                    data-control="select2" data-placeholder="All Supplier">
                    <option value="all">All Supplier</option>
                    @foreach ($suppliers as $supplier)
                        <option value="{{ $supplier->id }}">{{ $supplier->name }}</option>
                    @endforeach
                </select>
            </div>
            <div class="col-lg-2">
                <select id="filter_brand" class="form-control w-100 form-select-sm form-select"
                    data-control="select2" data-placeholder="All Brands">
                    <option value="all">All Brands</option>
                    @foreach ($brands as $brand)
                        <option value="{{ $brand->id }}">{{ $brand->name }}</option>
                    @endforeach
                </select>
            </div>
            <div class="col-lg-3">
                <div class="ms-0 ms-lg-3 my-5 my-md-0 text-center">
                    <a data-status="all"
                        class="search-status fs-7 text-dark bg-gray-200 rounded py-2 px-3 mx-1">All</a>
                    <a data-status="active"
                        class="search-status fs-7 text-dark rounded py-2 px-3 mx-1">Active</a>
                    <a data-status="draft" class="search-status fs-7 text-dark rounded py-2 px-3 mx-1">Draft</a>
                </div>
            </div>
            <div class="col-lg-2 text-end">
                <a class="btn btn-dark btn-sm ms-2" href="{{ route('products.sync') }}">
                    <i class="bi bi-download fs-4 me-1"></i>Import Products</a>
            </div>
        </div>
    </div>
    <div class="card-body">
        <div class="table-responsive">
            <table id="wc_products_table" class="table table-row-bordered gy-5 gs-7 border rounded">
                <thead>
                    <tr class="fw-semibold fs-6 text-gray-800 text-uppercase bg-light">
                        <th class="text-gray-700 fw-semibold fs-7 w-25 w-lg-50">Title</th>
                        <th class="text-gray-700 fw-semibold fs-7 min-w-100px">Supplier</th>
                        <th class="text-gray-700 fw-semibold fs-7 min-w-100px">SKU</th>
                        <th class="text-gray-700 fw-semibold fs-7 min-w-100px">Status</th>
                        <th class="text-gray-700 fw-semibold fs-7 min-w-100px">Price</th>
                        <th class="text-gray-700 fw-semibold fs-7 min-w-100px">Stock</th>
                    </tr>
                </thead>
                <tbody>
                    <!-- Data will be populated via Ajax -->
                </tbody>
            </table>
        </div>
    </div>
</div>
```

### 2. Backend: Fetch Product Data Using Ajax
Here is the JavaScript code that initializes DataTables and applies filters dynamically.

```javascript
var table = $("#wc_products_table").DataTable({
    processing: true,
    serverSide: true,
    ajax: {
        url: 'api/products/get',
        data: function(d) {
            d.keyword = $('#search_customer').val();
            d.supplier_id = $('#filter_supplier').val();
            d.brand_id = $('#filter_brand').val();
            d.status = $('.search-status.bg-gray-200').data('status') || 'all';
        }
    },
    columns: [{
            data: 'title',
            name: 'title',
            render: function(data, type, row) {
                let brandName = '';
                if (row.brand && row.brand !== undefined) {
                    brandName = row.brand.name;
                }

                return `
                    <div style="display: flex; align-items: center;">
                        <div style="margin-right: 10px;">
                            ${row.images && row.images.length > 0 ? 
                                `<img src="${row.images[row.images.length - 1].src}" alt="${data}" style="max-width: 50px; max-height: 50px;">` : 
                                `<img src="default_image_path.jpg" alt="No Image" style="max-width: 50px; max-height: 50px;">`}
                        </div>
                        <div>
                            <a href="products/show/${row.handle}" class="fw-medium text-dark">${row.title}</a><br>
                            <span class="text-gray-600 fs-7">${brandName}</span>
                        </div>
                    </div>
                `;
            }
        },
        {
            data: 'supplier.name',
            name: 'supplier.name',
            render: function(data, type, row) {
                return data && data !== undefined ? data : null;
            }
        },
        {
            data: 'sku',
            name: 'sku'
        },
        {
            data: 'status',
            name: 'status',
            render: function(data, type, row) {
                switch (data) {
                    case 'active':
                        return '<span class="badge badge-wc-active">Active</span>';
                    case 'draft':
                        return '<span class="badge badge-wc-draft">Draft</span>';
                    default:
                        return '<span class="badge badge-wc-archived text-capitalize">' +
                            data + '</span>';
                }
            }
        },
        {
            data: 'price_with_symbol',
            name: 'price_with_symbol'
        },
        {
            data: 'stocks',
            name: 'stocks',
            render: function(data) {
                return data.map(stock => stock.stock_count).reduce((a, b) => a + b, 0);
            }
        }
    ],
    pageLength: 10,
    lengthMenu: [10, 25, 50],
});
```

### 3. Ajax Filters Implementation
The following jQuery code is used to trigger filters and reload the data table.

``` javascript
let searchTimeout;

$("#search_customer").on("keyup", function() {
    const keyword = $(this).val();
    clearTimeout(searchTimeout);
    searchTimeout = setTimeout(function() {
        if (keyword.length >= 2 || keyword.length === 0) {
            table.ajax.reload();
        }
    }, 500);
});

$("#filter_brand").on("change", function() {
    table.ajax.reload();
});

$('#filter_supplier').on('change', function() {
    table.ajax.reload();
});

$('.search-status').on('click', function(e) {
    e.preventDefault();
    $('.search-status').removeClass('bg-gray-200');
    $(this).addClass('bg-gray-200');
    table.ajax.reload();
});
```


### 4. Backend: Fetching Products Data
In the backend, we handle product fetching and apply filters based on customer input.

#### Controller Code (for initial page load)

```php
public function index()
{
    try {
        $products = Product::where('store_id', Auth::user()->store_id)->get();
        $suppliers = Supplier::where('store_id', Auth::user()->store_id)->get();
        $brands = Brand::where('store_id', Auth::user()->store_id)->get();

        return view('store.products.index', compact('products', 'suppliers', 'brands'));
    } catch (\Exception $e) {
        \Log::error('Error fetching products: ' . $e->getMessage());
        return response()->view('errors.500', [], 500);
    }
}
```

#### Controller Code (for fetching products with filters)

```php
public function getProducts(Request $request)
{
    $query = Product::with(['supplier', 'brand','images']);

    if ($request->has('keyword')) {
        $query->where('title', 'like', '%' . $request->get('keyword') . '%');
    }

    if ($request->has('brand_id') && $request->brand_id != 'all') {
        $query->where('brand_id', $request->brand_id);
    }

    if ($request->has('supplier_id') && $request->supplier_id != 'all') {
        $query->where('supplier_id', $request->supplier_id);
    }

    // Filter by status if provided and not 'all'
    if ($request->has('status') && $request->status != 'all') {
        $query->where('status', $request->status);
    }

    // Calculate the current page based on the 'start' and 'length' parameters from DataTables
    $page = ($request->get('start', 0) / $request->get('length', 50)) + 1;

    // Paginate the results
    $products = $query->paginate($request->get('length', 50), ['*'], 'page', $page);

    // Return the products along with the total number of records
    return response()->json([
        'data' => $products->items(),
        'recordsTotal' => $products->total(),
        'recordsFiltered' => $products->total(),
    ]);
}
```

### Explanation of the continuation:

#### Supplier and Brand Filter:
If the request has supplier_id and it's not equal to 'all', the query filters products by supplier_id.
Similarly, for brand_id, if provided and not equal to 'all', products are filtered by the brand_id.

#### Status Filter:
The products are filtered by their status if the status is provided and not equal to 'all'.
Pagination:

The current page is calculated based on DataTables parameters (start and length), and the paginate() method is used to get paginated results.
JSON Response:

It returns a JSON response with product data, total records, and filtered records. This is used by DataTables to display the products on the frontend.
