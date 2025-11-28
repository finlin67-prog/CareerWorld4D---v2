# Career World 4D - Mapbox Implementation Guide

## Overview
Build an interactive city district map using Mapbox GL JS with your AI-generated district illustrations as custom map layers. This guide assumes you have a Mapbox API key and AI-generated district images.

---

## Phase 1: Setup & Configuration

### Install Dependencies
```bash
npm install mapbox-gl react-map-gl
npm install @turf/turf  # For geo calculations
```

### Project Structure
```
/src
  /components
    CareerMap.jsx           # Main map component
    BuildingMarker.jsx      # Individual role markers
    InfoPanel.jsx           # Details panel
    DistrictOverlay.jsx     # District illustrations overlay
  /data
    careerData.js           # Your career data
    districtCoordinates.js  # Geographic boundaries
  /assets
    /districts
      financial-district.png
      tech-valley.png
      enterprise-boulevard.png
      innovation-quarter.png
      central-hub.png
```

---

## Phase 2: Map Configuration

### Basic Mapbox Setup

```javascript
// CareerMap.jsx
import React, { useState, useRef, useEffect } from 'react';
import Map, { Source, Layer, Marker, Popup } from 'react-map-gl';
import 'mapbox-gl/dist/mapbox-gl.css';

const MAPBOX_TOKEN = 'your_mapbox_token_here';

// Define your "city" center - this can be anywhere, we'll use Miami
const CITY_CENTER = {
  longitude: -80.1918,
  latitude: 25.7617,
  zoom: 12
};

function CareerMap() {
  const [viewState, setViewState] = useState(CITY_CENTER);
  const [selectedBuilding, setSelectedBuilding] = useState(null);
  const mapRef = useRef();

  return (
    <div style={{ width: '100vw', height: '100vh' }}>
      <Map
        ref={mapRef}
        {...viewState}
        onMove={evt => setViewState(evt.viewState)}
        mapboxAccessToken={MAPBOX_TOKEN}
        mapStyle="mapbox://styles/mapbox/light-v11" // Clean base style
        style={{ width: '100%', height: '100%' }}
      >
        {/* District overlays and markers will go here */}
      </Map>
    </div>
  );
}

export default CareerMap;
```

---

## Phase 3: District Overlay System

### Strategy: Custom Image Overlays

We'll position your AI-generated district images as custom overlays on the map.

```javascript
// districtCoordinates.js
export const districts = {
  financial: {
    id: 'financial-district',
    name: 'Financial District',
    years: '2000-2006',
    color: '#0066cc',
    // Define corners of the district (top-left, top-right, bottom-right, bottom-left)
    coordinates: [
      [-80.20, 25.78],  // Northwest corner
      [-80.18, 25.78],  // Northeast corner
      [-80.18, 25.76],  // Southeast corner
      [-80.20, 25.76]   // Southwest corner
    ],
    imagePath: '/assets/districts/financial-district.png'
  },
  techValley: {
    id: 'tech-valley',
    name: 'Tech Valley',
    years: '2006-2014',
    color: '#9933ff',
    coordinates: [
      [-80.20, 25.76],
      [-80.18, 25.76],
      [-80.18, 25.74],
      [-80.20, 25.74]
    ],
    imagePath: '/assets/districts/tech-valley.png'
  },
  enterpriseBoulevard: {
    id: 'enterprise-boulevard',
    name: 'Enterprise Boulevard',
    years: '2014-2023',
    color: '#00bbff',
    coordinates: [
      [-80.18, 25.78],
      [-80.16, 25.78],
      [-80.16, 25.76],
      [-80.18, 25.76]
    ],
    imagePath: '/assets/districts/enterprise-boulevard.png'
  },
  innovationQuarter: {
    id: 'innovation-quarter',
    name: 'Innovation Quarter',
    years: '2023-2025',
    color: '#ff0066',
    coordinates: [
      [-80.18, 25.76],
      [-80.16, 25.76],
      [-80.16, 25.74],
      [-80.18, 25.74]
    ],
    imagePath: '/assets/districts/innovation-quarter.png'
  }
};
```

### Adding District Image Overlays

```javascript
// DistrictOverlay.jsx
import { Source, Layer } from 'react-map-gl';

function DistrictOverlay({ district }) {
  // Create GeoJSON for the district
  const districtGeoJSON = {
    type: 'Feature',
    geometry: {
      type: 'Polygon',
      coordinates: [district.coordinates]
    }
  };

  return (
    <>
      {/* Image overlay */}
      <Source
        id={`${district.id}-image`}
        type="image"
        url={district.imagePath}
        coordinates={district.coordinates}
      >
        <Layer
          id={`${district.id}-layer`}
          type="raster"
          paint={{
            'raster-opacity': 0.9,
            'raster-fade-duration': 300
          }}
        />
      </Source>

      {/* District border/highlight */}
      <Source
        id={`${district.id}-border`}
        type="geojson"
        data={districtGeoJSON}
      >
        <Layer
          id={`${district.id}-border-layer`}
          type="line"
          paint={{
            'line-color': district.color,
            'line-width': 3,
            'line-opacity': 0.6
          }}
        />
      </Source>
    </>
  );
}
```

---

## Phase 4: Building Markers (Your Career Roles)

### Career Data Structure

```javascript
// careerData.js
export const careerBuildings = [
  {
    id: 1,
    company: "Goldman Sachs",
    title: "Learning & Professional Development Analyst",
    years: "2000-2002",
    duration: 2.5,
    districtId: 'financial-district',
    coordinates: [-80.195, 25.77], // Position within district
    color: '#0066cc',
    achievements: [
      "Directed implementation of Enterprise LMS",
      "Managed all eLearning projects and reporting tools"
    ],
    disciplines: ["Operations", "Learning Systems"],
    technologies: ["LMS"],
    industry: "FinTech"
  },
  {
    id: 2,
    company: "Standard & Poor's",
    title: "Learning Management Systems Strategist",
    years: "2003-2005",
    duration: 2.0,
    districtId: 'financial-district',
    coordinates: [-80.190, 25.772],
    color: '#0066cc',
    achievements: [
      "Led implementation for LMS across business units",
      "Recognized by CEO for on-time, on-budget delivery"
    ],
    disciplines: ["Operations", "Training"],
    technologies: ["LMS"],
    industry: "FinTech"
  },
  // ... Add all 11 roles with their coordinates
  {
    id: 8,
    company: "Salesforce",
    title: "Global Marketing Operations Leader",
    years: "2014-2021",
    duration: 7.6,
    districtId: 'enterprise-boulevard',
    coordinates: [-80.172, 25.77], // Central prominent location
    color: '#00bbff',
    size: 'large', // Special marker for longest tenure
    achievements: [
      "76% YoY increase in marketing-sourced pipeline",
      "190% improvement in reporting delivery time",
      "150% increase in lead generation",
      "30% YoY pipeline increase through ABM"
    ],
    disciplines: ["Marketing Operations", "Demand Generation", "ABM", "Analytics"],
    technologies: ["Salesforce", "Marketo", "Tableau", "6sense"],
    industry: "SaaS"
  }
  // ... continue for all roles
];
```

### Building Marker Component

```javascript
// BuildingMarker.jsx
import { Marker } from 'react-map-gl';

function BuildingMarker({ building, isSelected, onClick, isHighlighted }) {
  // Size based on duration
  const getSize = () => {
    if (building.size === 'large') return 60;
    if (building.duration > 5) return 50;
    if (building.duration > 2) return 40;
    return 30;
  };

  const size = getSize();

  return (
    <Marker
      longitude={building.coordinates[0]}
      latitude={building.coordinates[1]}
      anchor="center"
      onClick={onClick}
    >
      <div
        style={{
          width: `${size}px`,
          height: `${size}px`,
          cursor: 'pointer',
          position: 'relative',
          transition: 'all 0.3s ease'
        }}
      >
        {/* Building icon/marker */}
        <div
          style={{
            width: '100%',
            height: '100%',
            backgroundColor: building.color,
            borderRadius: '8px',
            border: isSelected ? '4px solid white' : '2px solid rgba(255,255,255,0.5)',
            boxShadow: isHighlighted 
              ? `0 0 20px ${building.color}` 
              : '0 2px 8px rgba(0,0,0,0.3)',
            display: 'flex',
            alignItems: 'center',
            justifyContent: 'center',
            transform: isSelected ? 'scale(1.2)' : 'scale(1)',
            opacity: isHighlighted ? 1 : 0.85
          }}
        >
          {/* Building icon - could be company logo or simplified building shape */}
          <div style={{ 
            width: '60%', 
            height: '60%', 
            backgroundColor: 'rgba(255,255,255,0.3)',
            borderRadius: '4px'
          }} />
        </div>

        {/* Label */}
        {(isSelected || isHighlighted) && (
          <div
            style={{
              position: 'absolute',
              top: '100%',
              left: '50%',
              transform: 'translateX(-50%)',
              marginTop: '8px',
              backgroundColor: 'rgba(0,0,0,0.8)',
              color: 'white',
              padding: '4px 8px',
              borderRadius: '4px',
              fontSize: '12px',
              fontWeight: 'bold',
              whiteSpace: 'nowrap',
              pointerEvents: 'none'
            }}
          >
            {building.company}
          </div>
        )}
      </div>
    </Marker>
  );
}

export default BuildingMarker;
```

---

## Phase 5: Info Panel

```javascript
// InfoPanel.jsx
function InfoPanel({ building, onClose }) {
  if (!building) return null;

  return (
    <div
      style={{
        position: 'fixed',
        bottom: 0,
        left: 0,
        right: 0,
        backgroundColor: 'rgba(0, 0, 0, 0.95)',
        backdropFilter: 'blur(10px)',
        padding: '24px',
        maxHeight: '40vh',
        overflowY: 'auto',
        borderTop: `4px solid ${building.color}`,
        color: 'white',
        animation: 'slideUp 0.3s ease'
      }}
    >
      <div style={{ maxWidth: '1200px', margin: '0 auto' }}>
        <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'start' }}>
          <div>
            <h2 style={{ 
              color: building.color, 
              margin: '0 0 8px 0',
              fontSize: '28px'
            }}>
              {building.company}
            </h2>
            <p style={{ 
              fontSize: '18px', 
              color: '#00d4ff',
              margin: '0 0 4px 0'
            }}>
              {building.title}
            </p>
            <p style={{ 
              color: 'rgba(255,255,255,0.7)',
              margin: '0 0 16px 0'
            }}>
              {building.years} • {building.duration} years • {building.industry}
            </p>
          </div>
          <button
            onClick={onClose}
            style={{
              background: 'none',
              border: '2px solid rgba(255,255,255,0.3)',
              color: 'white',
              padding: '8px 16px',
              borderRadius: '4px',
              cursor: 'pointer',
              fontSize: '14px'
            }}
          >
            Close
          </button>
        </div>

        <div style={{ 
          display: 'grid', 
          gridTemplateColumns: '1fr 1fr 1fr', 
          gap: '24px',
          marginTop: '16px'
        }}>
          <div>
            <h3 style={{ color: '#ffd700', fontSize: '14px', marginBottom: '8px' }}>
              Key Achievements
            </h3>
            {building.achievements.map((achievement, idx) => (
              <div
                key={idx}
                style={{
                  backgroundColor: 'rgba(255, 215, 0, 0.1)',
                  borderLeft: '3px solid #ffd700',
                  padding: '8px 12px',
                  marginBottom: '8px',
                  borderRadius: '4px',
                  fontSize: '13px'
                }}
              >
                {achievement}
              </div>
            ))}
          </div>

          <div>
            <h3 style={{ color: '#00d4ff', fontSize: '14px', marginBottom: '8px' }}>
              Marketing Disciplines
            </h3>
            <div style={{ display: 'flex', flexWrap: 'wrap', gap: '8px' }}>
              {building.disciplines.map((discipline, idx) => (
                <span
                  key={idx}
                  style={{
                    backgroundColor: 'rgba(0, 212, 255, 0.2)',
                    border: '1px solid rgba(0, 212, 255, 0.4)',
                    padding: '4px 12px',
                    borderRadius: '16px',
                    fontSize: '12px'
                  }}
                >
                  {discipline}
                </span>
              ))}
            </div>
          </div>

          <div>
            <h3 style={{ color: '#9933ff', fontSize: '14px', marginBottom: '8px' }}>
              Technologies Used
            </h3>
            <div style={{ display: 'flex', flexWrap: 'wrap', gap: '8px' }}>
              {building.technologies.map((tech, idx) => (
                <span
                  key={idx}
                  style={{
                    backgroundColor: 'rgba(153, 51, 255, 0.2)',
                    border: '1px solid rgba(153, 51, 255, 0.4)',
                    padding: '4px 12px',
                    borderRadius: '16px',
                    fontSize: '12px'
                  }}
                >
                  {tech}
                </span>
              ))}
            </div>
          </div>
        </div>
      </div>
    </div>
  );
}

export default InfoPanel;
```

---

## Phase 6: Putting It All Together

```javascript
// CareerMap.jsx (Complete)
import React, { useState, useRef } from 'react';
import Map from 'react-map-gl';
import 'mapbox-gl/dist/mapbox-gl.css';
import { districts } from './data/districtCoordinates';
import { careerBuildings } from './data/careerData';
import DistrictOverlay from './components/DistrictOverlay';
import BuildingMarker from './components/BuildingMarker';
import InfoPanel from './components/InfoPanel';

const MAPBOX_TOKEN = 'your_mapbox_token_here';

function CareerMap() {
  const [viewState, setViewState] = useState({
    longitude: -80.1818,
    latitude: 25.76,
    zoom: 13
  });
  const [selectedBuilding, setSelectedBuilding] = useState(null);
  const [hoveredBuilding, setHoveredBuilding] = useState(null);
  const mapRef = useRef();

  const handleBuildingClick = (building) => {
    setSelectedBuilding(building);
    
    // Fly to building location
    mapRef.current?.flyTo({
      center: building.coordinates,
      zoom: 15,
      duration: 1000
    });
  };

  return (
    <div style={{ width: '100vw', height: '100vh', position: 'relative' }}>
      {/* Title */}
      <div
        style={{
          position: 'fixed',
          top: '20px',
          left: '50%',
          transform: 'translateX(-50%)',
          zIndex: 1000,
          textAlign: 'center'
        }}
      >
        <h1
          style={{
            fontSize: '48px',
            fontWeight: 'bold',
            background: 'linear-gradient(135deg, #00d4ff 0%, #7b2ff7 100%)',
            WebkitBackgroundClip: 'text',
            WebkitTextFillColor: 'transparent',
            margin: 0,
            textShadow: '0 0 30px rgba(0, 212, 255, 0.5)'
          }}
        >
          CAREER WORLD 4D
        </h1>
        <p style={{ color: 'white', margin: '8px 0 0 0', fontSize: '18px' }}>
          Michael Findling's Interactive Career Map
        </p>
      </div>

      {/* District Legend */}
      <div
        style={{
          position: 'fixed',
          top: '120px',
          right: '20px',
          zIndex: 1000,
          backgroundColor: 'rgba(0, 0, 0, 0.8)',
          backdropFilter: 'blur(10px)',
          padding: '16px',
          borderRadius: '8px',
          border: '2px solid rgba(255, 255, 255, 0.2)'
        }}
      >
        <h3 style={{ color: 'white', margin: '0 0 12px 0', fontSize: '14px' }}>
          Career Districts
        </h3>
        {Object.values(districts).map((district) => (
          <div
            key={district.id}
            style={{
              display: 'flex',
              alignItems: 'center',
              marginBottom: '8px',
              cursor: 'pointer'
            }}
            onClick={() => {
              const centerLng = (district.coordinates[0][0] + district.coordinates[2][0]) / 2;
              const centerLat = (district.coordinates[0][1] + district.coordinates[2][1]) / 2;
              mapRef.current?.flyTo({
                center: [centerLng, centerLat],
                zoom: 14,
                duration: 1000
              });
            }}
          >
            <div
              style={{
                width: '16px',
                height: '16px',
                backgroundColor: district.color,
                borderRadius: '4px',
                marginRight: '8px'
              }}
            />
            <div>
              <div style={{ color: 'white', fontSize: '12px', fontWeight: 'bold' }}>
                {district.name}
              </div>
              <div style={{ color: 'rgba(255,255,255,0.6)', fontSize: '10px' }}>
                {district.years}
              </div>
            </div>
          </div>
        ))}
      </div>

      {/* Map */}
      <Map
        ref={mapRef}
        {...viewState}
        onMove={evt => setViewState(evt.viewState)}
        mapboxAccessToken={MAPBOX_TOKEN}
        mapStyle="mapbox://styles/mapbox/dark-v11"
        style={{ width: '100%', height: '100%' }}
      >
        {/* District overlays */}
        {Object.values(districts).map((district) => (
          <DistrictOverlay key={district.id} district={district} />
        ))}

        {/* Building markers */}
        {careerBuildings.map((building) => (
          <BuildingMarker
            key={building.id}
            building={building}
            isSelected={selectedBuilding?.id === building.id}
            isHighlighted={hoveredBuilding?.id === building.id}
            onClick={() => handleBuildingClick(building)}
          />
        ))}
      </Map>

      {/* Info Panel */}
      <InfoPanel
        building={selectedBuilding}
        onClose={() => setSelectedBuilding(null)}
      />
    </div>
  );
}

export default CareerMap;
```

---

## Phase 7: Next Steps

### 1. Generate Your District Images
Use the AI image generation prompts I provided earlier with PicLumen or Nano Banana

### 2. Adjust Coordinates
Fine-tune the district coordinates and building positions based on your actual generated images

### 3. Add Polish
- Smooth zoom transitions
- Category filters (by industry, by discipline, by technology)
- Search functionality
- Mobile responsiveness

### 4. Deploy
- Host on Vercel, Netlify, or GitHub Pages
- Your Mapbox usage should stay within free tier for portfolio use

---

## Tips for Success

1. **Start Simple:** Get one district working first, then add the others
2. **Test Zoom Levels:** Make sure your images look good at different zoom levels
3. **Performance:** Use proper image optimization (WebP format, appropriate resolution)
4. **Fallback:** If AI images don't work out, use colored shapes with labels as placeholders
5. **Iterate:** This is easier to refine than the abstract mountain concept

This approach gives you the professional polish of the golf course map with the creative flair of Universal Studios!
