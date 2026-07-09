module.exports = async function handler(req, res) {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'POST, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type');

  if (req.method === 'OPTIONS') return res.status(200).end();
  if (req.method !== 'POST') return res.status(405).json({ error: 'POST only' });

  try {
    const { image } = req.body;
    if (!image) return res.status(400).json({ error: 'No image provided' });

    const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer ' + process.env.OPENROUTER_API_KEY
      },
      body: JSON.stringify({
        model: 'google/gemini-2.0-flash-001',
        messages: [{
          role: 'user',
          content: [
            { type: 'text', text: 'Extract table data from this image. Return JSON with "headings" (array of column header strings) and "columns" (array of arrays, each containing the cell values for that column). Only return the JSON, no markdown or explanation.' },
            { type: 'image_url', image_url: { url: 'data:image/jpeg;base64,' + image } }
          ]
        }]
      })
    });

    const data = await response.json();
    if (!response.ok) return res.status(500).json({ error: 'API error: ' + (data.error?.message || JSON.stringify(data.error) || response.status) });

    const text = data.choices?.[0]?.message?.content || '';
    const clean = text.replace(/```json\s*/g, '').replace(/```\s*/g, '').trim();

    try {
      return res.status(200).json({ success: true, data: JSON.parse(clean) });
    } catch (e) {
      return res.status(200).json({ success: false, data: { raw: text } });
    }
  } catch (err) {
    return res.status(500).json({ error: String(err.message || err) });
  }
};
