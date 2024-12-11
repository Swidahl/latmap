import React, { useState } from 'react';
import { MapContainer, TileLayer, Marker, Popup, useMapEvents } from 'react-leaflet';
import 'leaflet/dist/leaflet.css';
import L from 'leaflet';

// Mock data - in real app, this would come from a comprehensive geographical database
const worldCities = [
  { name: 'Aarhus', country: 'Denmark', latitude: 56.1629, longitude: 10.2033 },
  { name: 'Buenos Aires', country: 'Argentina', latitude: -34.6037, longitude: -58.3816 },
  { name: 'Tokyo', country: 'Japan', latitude: 35.6762, longitude: 139.6503 },
  { name: 'New York', country: 'United States', latitude: 40.7128, longitude: -74.0060 },
  { name: 'Cape Town', country: 'South Africa', latitude: -33.9249, longitude: 18.4241 },
  { name: 'Sydney', country: 'Australia', latitude: -33.8688, longitude: 151.2093 },
];

const GeoLongitudeApp = () => {
  const [selectedLocation, setSelectedLocation] = useState(null);
  const [longitudalCities, setLongitudinalCities] = useState([]);

  const findLongitudinalCities = (longitude) => {
    // Allow some tolerance (e.g., ±0.5 degrees) for matching longitude
    return worldCities.filter(city => 
      Math.abs(city.longitude - longitude) < 0.5 && 
      city.name !== selectedLocation.name
    );
  };

  const LocationSelector = () => {
    useMapEvents({
      click: (e) => {
        const { lat, lng } = e.latlng;
        const closestCity = worldCities.reduce((prev, curr) => 
          Math.abs(curr.latitude - lat) < Math.abs(prev.latitude - lat) ? curr : prev
        );
        
        setSelectedLocation(closestCity);
        const matchingCities = findLongitudinalCities(closestCity.longitude);
        setLongitudinalCities(matchingCities);
      }
    });
    return null;
  };

  return (
    <div className="flex h-screen">
      <div className="w-3/4">
        <MapContainer 
          center={[0, 0]} 
          zoom={2} 
          style={{ height: '100%', width: '100%' }}
        >
          <TileLayer
            url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
            attribution='&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
          />
          <LocationSelector />
          
          {selectedLocation && (
            <Marker 
              position={[selectedLocation.latitude, selectedLocation.longitude]}
              icon={L.divIcon({ 
                className: 'bg-blue-500 rounded-full w-6 h-6 flex items-center justify-center',
                html: `<div class="text-white font-bold">🌍</div>` 
              })}
            >
              <Popup>
                Selected Location: {selectedLocation.name}, {selectedLocation.country}
              </Popup>
            </Marker>
          )}

          {longitudalCities.map((city, index) => (
            <Marker 
              key={index}
              position={[city.latitude, city.longitude]}
              icon={L.divIcon({ 
                className: 'bg-red-500 rounded-full w-6 h-6 flex items-center justify-center',
                html: `<div class="text-white font-bold">🏙️</div>` 
              })}
            >
              <Popup>
                {city.name}, {city.country}
              </Popup>
            </Marker>
          ))}
        </MapContainer>
      </div>
      
      <div className="w-1/4 p-4 overflow-y-auto bg-gray-100">
        <h2 className="text-xl font-bold mb-4">Longitudinal Cities</h2>
        {selectedLocation ? (
          <>
            <p>Base Location: {selectedLocation.name}, {selectedLocation.country}</p>
            <p>Longitude: {selectedLocation.longitude.toFixed(4)}°</p>
            <h3 className="mt-4 font-semibold">Matching Cities:</h3>
            {longitudalCities.length > 0 ? (
              <ul>
                {longitudalCities.map((city, index) => (
                  <li key={index} className="mb-2">
                    {city.name}, {city.country} 
                    <span className="text-gray-500 ml-2">
                      (Lat: {city.latitude.toFixed(4)}°)
                    </span>
                  </li>
                ))}
              </ul>
            ) : (
              <p>No matching cities found.</p>
            )}
          </>
        ) : (
          <p>Click on the map to select a location.</p>
        )}
      </div>
    </div>
  );
};

export default GeoLongitudeApp;
