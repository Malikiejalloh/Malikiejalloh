const mongoose = require('mongoose');

const productSchema = mongoose.Schema({
    name: { type: String, required: true },
    category: { type: String },
    sku: { type: String },
});

module.exports = mongoose.model('Product', productSchema);
const mongoose = require('mongoose');

const priceNormSchema = mongoose.Schema({
    product_id: { type: mongoose.Schema.Types.ObjectId, ref: 'Product', required: true },
    norm_price: { type: Number, required: true },
    currency: { type: String, default: 'USD' },
    region: { type: String }, // Could be based on location, if applicable
    updatedAt: { type: Date, default: Date.now },
});

module.exports = mongoose.model('PriceNorm', priceNormSchema);
const express = require('express');
const router = express.Router();
const PriceNorm = require('../models/priceNorm');
const Product = require('../models/product'); // To validate if the product exists

// Add a new price norm
router.post('/', async (req, res) => {
    const { product_id, norm_price, currency, region } = req.body;
    try {
        const productExists = await Product.findById(product_id);
        if (!productExists) {
            return res.status(400).json({ message: 'Product not found' });
        }

        const priceNorm = new PriceNorm({ product_id, norm_price, currency, region });
        await priceNorm.save();
        res.status(201).json(priceNorm);
    } catch (error) {
        res.status(400).json({ message: error.message });
    }
});

// Get all price norms
router.get('/', async (req, res) => {
    try {
        const priceNorms = await PriceNorm.find().populate('product_id');
        res.status(200).json(priceNorms);
    } catch (error) {
        res.status(500).json({ message: error.message });
    }
});

// Get price norm by product
router.get('/:product_id', async (req, res) => {
    try {
        const priceNorm = await PriceNorm.findOne({ product_id: req.params.product_id }).populate('product_id');
        if (!priceNorm) {
            return res.status(404).json({ message: 'Price norm not found' });
        }
        res.status(200).json(priceNorm);
    } catch (error) {
        res.status(500).json({ message: error.message });
    }
});

// Update price norm
router.put('/:id', async (req, res) => {
    const { norm_price, currency, region } = req.body;
    try {
        const updatedNorm = await PriceNorm.findByIdAndUpdate(
            req.params.id, 
            { norm_price, currency, region, updatedAt: Date.now() }, 
            { new: true }
        );
        res.status(200).json(updatedNorm);
    } catch (error) {
        res.status(400).json({ message: error.message });
    }
});

// Delete price norm
router.delete('/:id', async (req, res) => {
    try {
        await PriceNorm.findByIdAndDelete(req.params.id);
        res.status(200).json({ message: 'Price norm deleted' });
    } catch (error) {
        res.status(400).json({ message: error.message });
    }
});const priceNormRoutes = require('./routes/priceNormRoutes');
app.use('/api/priceNorms', priceNormRoutes);
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const PriceNorms = () => {
    const [priceNorms, setPriceNorms] = useState([]);

    useEffect(() => {
        // Fetch price norms from the API
        axios.get('/api/priceNorms')
            .then(response => {
                setPriceNorms(response.data);
            })
            .catch(error => {
                console.error('There was an error fetching price norms!', error);
            });
    }, []);

    return (
        <div>
            <h2>Price Norms</h2>
            <table>
                <thead>
                    <tr>
                        <th>Product Name</th>
                        <th>Norm Price</th>
                        <th>Currency</th>
                        <th>Region</th>
                        <th>Last Updated</th>
                    </tr>
                </thead>
                <tbody>
                    {priceNorms.map(norm => (
                        <tr key={norm._id}>
                            <td>{norm.product_id.name}</td>
                            <td>{norm.norm_price}</td>
                            <td>{norm.currency}</td>
                            <td>{norm.region}</td>
                            <td>{new Date(norm.updatedAt).toLocaleString()}</td>
                        </tr>
                    ))}
                </tbody>
            </table>
        </div>
    );
};

export default PriceNorms;
import React from 'react';
import PriceNorms from './pages/PriceNorms';

const App = () => {
    return (
        <div>
            <PriceNorms />
        </div>
    );
};

export default App;
{
    "product_id": "product_object_id",
    "norm_price": 100,
    "currency": "USD",
    "region": "US"
}
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const PriceNorms = () => {
    const [priceNorms, setPriceNorms] = useState([]);
    const [editing, setEditing] = useState(false);
    const [editData, setEditData] = useState({
        id: '',
        norm_price: '',
        currency: '',
        region: ''
    });

    useEffect(() => {
        // Fetch price norms from the API
        axios.get('/api/priceNorms')
            .then(response => {
                setPriceNorms(response.data);
            })
            .catch(error => {
                console.error('There was an error fetching price norms!', error);
            });
    }, []);

    // Handle edit button click
    const handleEditClick = (priceNorm) => {
        setEditing(true);
        setEditData({
            id: priceNorm._id,
            norm_price: priceNorm.norm_price,
            currency: priceNorm.currency,
            region: priceNorm.region
        });
    };

    // Handle form field changes
    const handleInputChange = (e) => {
        const { name, value } = e.target;
        setEditData(prevState => ({
            ...prevState,
            [name]: value
        }));
    };

    // Submit the update form
    const handleSubmit = (e) => {
        e.preventDefault();
        axios.put(`/api/priceNorms/${editData.id}`, {
            norm_price: editData.norm_price,
            currency: editData.currency,
            region: editData.region
        })
        .then(response => {
            // Update the price norms list
            const updatedNorms = priceNorms.map(norm => 
                norm._id === editData.id ? response.data : norm
            );
            setPriceNorms(updatedNorms);
            setEditing(false);
            setEditData({ id: '', norm_price: '', currency: '', region: '' });
        })
        .catch(error => {
            console.error('There was an error updating the price norm!', error);
        });
    };

    return (
        <div>
            <h2>Price Norms</h2>
            <table>
                <thead>
                    <tr>
                        <th>Product Name</th>
                        <th>Norm Price</th>
                        <th>Currency</th>
                        <th>Region</th>
                        <th>Last Updated</th>
                        <th>Actions</th>
                    </tr>
                </thead>
                <tbody>
                    {priceNorms.map(norm => (
                        <tr key={norm._id}>
                            <td>{norm.product_id.name}</td>
                            <td>{norm.norm_price}</td>
                            <td>{norm.currency}</td>
                            <td>{norm.region}</td>
                            <td>{new Date(norm.updatedAt).toLocaleString()}</td>
                            <td>
                                <button onClick={() => handleEditClick(norm)}>Edit</button>
                            </td>
                        </tr>
                    ))}
                </tbody>
            </table>

            {/* Edit Price Norm Form */}
            {editing && (
                <div>
                    <h3>Edit Price Norm</h3>
                    <form onSubmit={handleSubmit}>
                        <label>Price</label>
                        <input 
                            type="number" 
                            name="norm_price" 
                            value={editData.norm_price} 
                            onChange={handleInputChange}
                            required 
                        />
                        <label>Currency</label>
                        <input 
                            type="text" 
                            name="currency" 
                            value={editData.currency} 
                            onChange={handleInputChange}
                            required 
                        />
                        <label>Region</label>
                        <input 
                            type="text" 
                            name="region" 
                            value={editData.region} 
                            onChange={handleInputChange}
                            required 
                        />
                        <button type="submit">Save Changes</button>
                        <button type="button" onClick={() => setEditing(false)}>Cancel</button>
                    </form>
                </div>
            )}
        </div>
    );
};

export default PriceNorms;
// Handle delete button click
const handleDeleteClick = (id) => {
    if (window.confirm("Are you sure you want to delete this price norm?")) {
        axios.delete(`/api/priceNorms/${id}`)
            .then(response => {
                // Remove deleted price norm from the list
                const filteredNorms = priceNorms.filter(norm => norm._id !== id);
                setPriceNorms(filteredNorms);
            })
            .catch(error => {
                console.error('There was an error deleting the price norm!', error);
            });
    }
};

// Add Delete button to table
<td>
    <button onClick={() => handleDeleteClick(norm._id)}>Delete</button>
</td>
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const PriceNorms = () => {
    const [priceNorms, setPriceNorms] = useState([]);
    const [searchTerm, setSearchTerm] = useState('');
    const [selectedCurrency, setSelectedCurrency] = useState('');
    const [selectedRegion, setSelectedRegion] = useState('');
    const [editing, setEditing] = useState(false);
    const [editData, setEditData] = useState({
        id: '',
        norm_price: '',
        currency: '',
        region: ''
    });

    useEffect(() => {
        // Fetch price norms from the API
        axios.get('/api/priceNorms')
            .then(response => {
                setPriceNorms(response.data);
            })
            .catch(error => {
                console.error('There was an error fetching price norms!', error);
            });
    }, []);

    // Handle input change for search and filters
    const handleSearchChange = (e) => {
        setSearchTerm(e.target.value);
    };

    const handleFilterChange = (e) => {
        const { name, value } = e.target;
        if (name === 'currency') {
            setSelectedCurrency(value);
        } else if (name === 'region') {
            setSelectedRegion(value);
        }
    };

    // Filter and search price norms
    const filteredNorms = priceNorms.filter(norm => {
        const matchesSearch = norm.product_id.name.toLowerCase().includes(searchTerm.toLowerCase());
        const matchesCurrency = selectedCurrency ? norm.currency === selectedCurrency : true;
        const matchesRegion = selectedRegion ? norm.region === selectedRegion : true;
        return matchesSearch && matchesCurrency && matchesRegion;
    });

    // Handle edit, delete, and update (same as previous code)

    return (
        <div>
            <h2>Price Norms</h2>
            
            {/* Search Bar */}
            <div>
                <input 
                    type="text" 
                    placeholder="Search by product name..." 
                    value={searchTerm} 
                    onChange={handleSearchChange} 
                />
            </div>

            {/* Filters */}
            <div>
                <label>Filter by Currency:</label>
                <select name="currency" value={selectedCurrency} onChange={handleFilterChange}>
                    <option value="">All</option>
                    <option value="USD">USD</option>
                    <option value="EUR">EUR</option>
                    {/* Add more currencies here */}
                </select>
                
                <label>Filter by Region:</label>
                <select name="region" value={selectedRegion} onChange={handleFilterChange}>
                    <option value="">All</option>
                    <option value="US">US</option>
                    <option value="EU">EU</option>
                    {/* Add more regions here */}
                </select>
            </div>

            {/* Price Norm Table */}
            <table>
                <thead>
                    <tr>
                        <th>Product Name</th>
                        <th>Norm Price</th>
                        <th>Currency</th>
                        <th>Region</th>
                        <th>Last Updated</th>
                        <th>Actions</th>
                    </tr>
                </thead>
                <tbody>
                    {filteredNorms.map(norm => (
                        <tr key={norm._id}>
                            <td>{norm.product_id.name}</td>
                            <td>{norm.norm_price}</td>
                            <td>{norm.currency}</td>
                            <td>{norm.region}</td>
                            <td>{new Date(norm.updatedAt).toLocaleString()}</td>
                            <td>
                                <button onClick={() => handleEditClick(norm)}>Edit</button>
                                <button onClick={() => handleDeleteClick(norm._id)}>Delete</button>
                            </td>
                        </tr>
                    ))}
                </tbody>
            </table>

            {/* Edit Form */}
            {editing && (
                <div>
                    <h3>Edit Price Norm</h3>
                    <form onSubmit={handleSubmit}>
                        <label>Price</label>
                        <input 
                            type="number" 
                            name="norm_price" 
                            value={editData.norm_price} 
                            onChange={handleInputChange}
                            required 
                        />
                        <label>Currency</label>
                        <input 
                            type="text" 
                            name="currency" 
                            value={editData.currency} 
                            onChange={handleInputChange}
                            required 
                        />
                        <label>Region</label>
                        <input 
                            type="text" 
                            name="region" 
                            value={editData.region} 
                            onChange={handleInputChange}
                            required 
                        />
                        <button type="submit">Save Changes</button>
                        <button type="button" onClick={() => setEditing(false)}>Cancel</button>
                    </form>
                </div>
            )}
        </div>
    );
};

export default PriceNorms;
heroku create your-app-name
git add .
git commit -m "Deploying app"
git push heroku master


module.exports = router;
