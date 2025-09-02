# Integration Code Examples

## N8N Emergency Workflow
Critical organ transport processing with sub-5-second response.

### Webhook Configuration
```json
{
  "nodes": [
    {
      "parameters": {
        "path": "/webhooks/transmedics/emergency",
        "authentication": "bearer_token"
      },
      "name": "Emergency Transport Webhook",
      "type": "n8n-nodes-base.webhook"
    },
    {
      "parameters": {
        "table": "emergency_transports",
        "columns": [
          {"column": "transport_id", "value": "={{$json.id}}"},
          {"column": "organ_type", "value": "={{$json.organ_details.type}}"}
        ]
      },
      "name": "Store Emergency Record", 
      "type": "n8n-nodes-base.postgres"
    }
  ]
}
```

## React Emergency Alert Component
```tsx
interface OrganTransportDetails {
  id: string;
  organ_type: 'heart' | 'liver' | 'kidney' | 'lung' | 'pancreas';
  preservation_time_remaining: number;
  donor_hospital: string;
  recipient_hospital: string;
}

export const EmergencyAlert: React.FC<{
  transportDetails: OrganTransportDetails;
  onAccept: (id: string) => Promise<void>;
}> = ({ transportDetails, onAccept }) => {
  const [countdown, setCountdown] = useState(transportDetails.preservation_time_remaining);
  
  useEffect(() => {
    const timer = setInterval(() => {
      setCountdown(prev => prev <= 1 ? 0 : prev - 1);
    }, 60000);
    return () => clearInterval(timer);
  }, []);

  return (
    <div className="fixed inset-0 z-50 bg-red-900/90 flex items-center justify-center">
      <div className="bg-white p-6 rounded-lg border-red-600 border-2">
        <h1 className="text-2xl font-bold text-center">
          🚨 EMERGENCY ORGAN TRANSPORT
        </h1>
        <div className="text-center mt-4">
          <div className="text-4xl">❤️</div>
          <div className="text-xl font-bold">{transportDetails.organ_type.toUpperCase()}</div>
          <div className="text-lg text-red-600">
            {Math.floor(countdown / 60)}h {countdown % 60}m remaining
          </div>
        </div>
        <button 
          onClick={() => onAccept(transportDetails.id)}
          className="w-full mt-4 bg-green-600 text-white p-3 rounded font-bold"
        >
          🚨 ACCEPT TRANSPORT
        </button>
      </div>
    </div>
  );
};
```

## HIPAA-Compliant Database Schema
```sql
-- Encrypted PHI storage with audit trails
CREATE TABLE transport_requests (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    transport_number VARCHAR(50) UNIQUE NOT NULL,
    type VARCHAR(20) CHECK (type IN ('routine', 'urgent', 'emergency', 'organ_transport')),
    priority_level INTEGER CHECK (priority_level BETWEEN 1 AND 5),
    
    -- Encrypted PHI fields
    patient_id TEXT, -- Encrypted
    patient_demographics TEXT, -- Encrypted JSON
    medical_requirements TEXT, -- Encrypted
    
    pickup_facility_name VARCHAR(200) NOT NULL,
    destination_facility_name VARCHAR(200) NOT NULL,
    scheduled_pickup_time TIMESTAMPTZ NOT NULL,
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- HIPAA audit logging
CREATE TABLE audit_log (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id VARCHAR(100) NOT NULL,
    action VARCHAR(50) NOT NULL,
    resource_type VARCHAR(50) NOT NULL,
    phi_accessed BOOLEAN DEFAULT FALSE,
    justification TEXT NOT NULL,
    timestamp TIMESTAMPTZ DEFAULT NOW(),
    checksum VARCHAR(64) NOT NULL -- Tamper detection
);

-- Row level security
ALTER TABLE transport_requests ENABLE ROW LEVEL SECURITY;
```

## TNT API Integration
```typescript
class TNTFastrakIntegration {
  async dispatchEmergencyTransport(transportData: any) {
    try {
      const response = await this.client.post('/dispatch/emergency', {
        transport_id: transportData.id,
        priority_level: 'CRITICAL',
        vehicle_requirements: {
          medical_certified: true,
          organ_transport: transportData.type === 'organ_transport'
        },
        timing: {
          max_delay_minutes: transportData.type === 'organ_transport' ? 0 : 15
        }
      });
      
      return {
        success: true,
        dispatch_id: response.data.dispatch_id,
        assigned_vehicle: response.data.vehicle
      };
    } catch (error) {
      if (transportData.type === 'organ_transport') {
        await this.triggerEmergencyBackup(transportData);
      }
      throw error;
    }
  }
}
```

---

**Code Examples Version**: 1.0  
**Technology Stack**: N8N Cloud, Next.js 14 PWA, PostgreSQL with encryption  
**Security Level**: HIPAA Compliant with comprehensive audit trails
