// Node.js Netlify Function. QOY: process.env.AVIATIONSTACK_KEY
const fetch = require("node-fetch");

exports.handler = async function(event) {
  const API_KEY = process.env.AVIATIONSTACK_KEY;
  if(!API_KEY) {
    return { statusCode: 500, body: JSON.stringify({ error: "Serverdə API açarı konfiqurasiya edilməyib." }) };
  }

  const params = event.queryStringParameters || {};
  const type = params.type || "all";
  const airport = params.airport || "GYD";

  let apiUrl = `http://api.aviationstack.com/v1/flights?access_key=${API_KEY}`;
  if(type === "arrivals") apiUrl += `&arr_iata=${airport}`;
  else if(type === "departures") apiUrl += `&dep_iata=${airport}`;
  else apiUrl += `&arr_iata=${airport}&dep_iata=${airport}`;

  try {
    const r = await fetch(apiUrl);
    if(!r.ok) {
      const text = await r.text();
      return { statusCode: 502, body: JSON.stringify({ error: "API error", detail: text }) };
    }
    const data = await r.json();
    const flights = (data.data || []).map(f => ({
      flight: f.flight ? (f.flight.iata || f.flight.number || "") : "",
      airline: f.airline ? f.airline.name : "",
      origin: f.departure ? (f.departure.iata || f.departure.airport) : "",
      destination: f.arrival ? (f.arrival.iata || f.arrival.airport) : "",
      scheduled: f.departure ? (f.departure.scheduled || null) : null,
      estimated: f.arrival ? (f.arrival.estimated || null) : null,
      status: f.flight_status || "",
      terminal: (f.departure && f.departure.terminal) || (f.arrival && f.arrival.terminal) || "",
      gate: (f.departure && f.departure.gate) || (f.arrival && f.arrival.gate) || ""
    }));

    return {
      statusCode: 200,
      body: JSON.stringify({ flights })
    };
  } catch (err) {
    return { statusCode: 500, body: JSON.stringify({ error: err.message }) };
  }
};
