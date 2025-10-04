// This is our serverless function. It acts as a secure proxy.
// It receives a request from our frontend, calls the Google AI API with the secret key,
// and then sends the response back to the frontend.

export default async function handler(req, res) {
    // Only allow POST requests
    if (req.method !== 'POST') {
        return res.status(405).json({ error: 'Method Not Allowed' });
    }

    try {
        const { query } = req.body;
        if (!query) {
            return res.status(400).json({ error: 'Search query is required.' });
        }

        // IMPORTANT: Get the API key from environment variables (set in Vercel)
        const API_KEY = process.env.GEMINI_API_KEY;
        if (!API_KEY) {
            return res.status(500).json({ error: 'API key is not configured on the server.' });
        }
        
        const GOOGLE_API_URL = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${API_KEY}`;
        
        const systemPrompt = `You are an expert material scientist specializing in polymers. 
        For a given product, identify the single most suitable plastic material and its common grade. 
        Then, list its key advantages and disadvantages for that specific application.
        Respond ONLY with a JSON object in the following format, with no extra text or explanations. The response must be in Persian:
        {
          "material": "نام ماده (فرمول شیمیایی)",
          "grade": "نام گرید رایج",
          "advantages": ["مزیت اول", "مزیت دوم", "..."],
          "disadvantages": ["عیب اول", "عیب دوم", "..."]
        }`;
        
        const payload = {
            contents: [{ parts: [{ text: `Product: "${query}"` }] }],
            systemInstruction: { parts: [{ text: systemPrompt }] },
            generationConfig: {
                responseMimeType: "application/json",
            }
        };

        const googleResponse = await fetch(GOOGLE_API_URL, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(payload),
        });

        if (!googleResponse.ok) {
            console.error('Google API Error:', await googleResponse.text());
            return res.status(googleResponse.status).json({ error: 'Failed to get a response from the AI model.' });
        }

        const result = await googleResponse.json();
        const jsonText = result.candidates[0].content.parts[0].text;
        const data = JSON.parse(jsonText);

        // Send the final data back to the browser
        res.status(200).json(data);

    } catch (error) {
        console.error('Server-side Error:', error);
        res.status(500).json({ error: 'An internal server error occurred.' });
    }
}
