<!DOCTYPE html>
<html lang="az">
<head>
  <meta charset="UTF-8" />
  <title>Bakı H.Əliyev Aeroportu Uçuşları</title>
  <style>
    body { font-family: Arial, sans-serif; padding: 20px; background: #f0f0f0; }
    h1 { color: #0066cc; }
    table { border-collapse: collapse; width: 100%; background: white; }
    th, td { border: 1px solid #ddd; padding: 8px; text-align: center; }
    th { background: #0066cc; color: white; }
  </style>
</head>
<body>
  <h1>Bakı Heydər Əliyev Aeroportu Uçuşları</h1>
  <table id="flights-table">
    <thead>
      <tr>
        <th>Uçuş Nömrəsi</th>
        <th>Şirkət</th>
        <th>Gediş</th>
        <th>Gəliş</th>
        <th>Status</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>

  <script>
    const API_KEY = 'SENIN_API_ACARIN'; // Buraya aviationstack API açarını yaz
    const airportCode = 'GYD'; // Bakı Heydər Əliyev aeroportu ICAO/IATA kodu

    async function fetchFlights() {
      try {
        // Uçuşların gələnlər (arrivals) və gedənlər (departures) sorğusu
        // aviationstack API-da endpointlər fərqlidir, biz burda gəlişləri nümunə göstəririk
        const url = `http://api.aviationstack.com/v1/flights?access_key=${API_KEY}&arr_icao=${airportCode}&limit=10`;

        const response = await fetch(url);
        const data = await response.json();

        const tbody = document.querySelector('#flights-table tbody');
        tbody.innerHTML = '';

        if (!data.data || data.data.length === 0) {
          tbody.innerHTML = '<tr><td colspan="5">Uçuş məlumatı tapılmadı.</td></tr>';
          return;
        }

        data.data.forEach(flight => {
          const row = document.createElement('tr');

          const flightNumber = flight.flight.iata || '—';
          const airline = flight.airline.name || '—';
          const departure = flight.departure ? flight.departure.airport : '—';
          const arrival = flight.arrival ? flight.arrival.airport : '—';
          const status = flight.flight_status || '—';

          row.innerHTML = `
            <td>${flightNumber}</td>
            <td>${airline}</td>
            <td>${departure}</td>
            <td>${arrival}</td>
            <td>${status}</td>
          `;

          tbody.appendChild(row);
        });

      } catch (error) {
        console.error('Xəta baş verdi:', error);
      }
    }

    fetchFlights();
  </script>
</body>
</html>
