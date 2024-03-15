# EZProxyWorkers
A Proxy for OpenAI API on Cloudflare Workers

Copy The code and implement the API_KEYs
```
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request));
});

const OPENAI_API_URL = 'https://api.openai.com/v1';
const NEW_API_KEY = 'YOUR_NEW_API_KEY';

async function handleRequest(request) {
  // Handle CORS preflight request
  if (request.method === 'OPTIONS') {
    return handleCorsPreflight(request);
  }

  // Log the full request details
  await logRequest(request);

  // Extract the path from the original request
  const originalUrl = new URL(request.url);
  const originalPath = originalUrl.pathname;

  // Construct the new URL for forwarding the request
  const forwardUrl = OPENAI_API_URL + originalPath;

  // Create a new request with the necessary headers
  let newHeaders = new Headers();
  newHeaders.set('Authorization', `Bearer ${NEW_API_KEY}`);

  // List of headers to copy from the original request
  const headersToCopy = ['content-Type', 'openai-beta'];

  // Copy headers from the original request
  copyHeaders(request, newHeaders, headersToCopy);

  const requestBody = request.method !== 'GET' ? await request.clone().text() : null;

  const newRequest = new Request(forwardUrl, {
    method: request.method,
    headers: newHeaders,
    body: requestBody
  });

  await logRequest(newRequest);

  // Forward the new request
  let response = await fetch(newRequest);

  await logRequest(response);

  // Return the response from the OpenAI API
  return response;
}

async function handleCorsPreflight(request) {
  const origin = request.headers.get('Origin');
  const accessControlRequestMethod = request.headers.get('Access-Control-Request-Method');
  const accessControlRequestHeaders = request.headers.get('Access-Control-Request-Headers');

  const responseHeaders = {
    'Access-Control-Allow-Origin': origin,
    'Access-Control-Allow-Methods': accessControlRequestMethod,
    'Access-Control-Allow-Headers': accessControlRequestHeaders,
    'Access-Control-Max-Age': '86400', // Cache the preflight request for 1 day
  };

  return new Response(null, {
    headers: responseHeaders,
    status: 204, // No Content
  });
}

async function logRequest(request) {
  console.log('Request Method:', request.method);
  console.log('Request URL:', request.url);

  console.log('Request Headers:');
  for (const [key, value] of request.headers.entries()) {
    console.log(`  ${key}: ${value}`);
  }

  const requestBody = await request.clone().text();
  console.log('Request Body:', requestBody);
}

function copyHeaders(originalRequest, newHeaders, headersToCopy) {
  headersToCopy.forEach(headerName => {
    const headerValue = originalRequest.headers.get(headerName);
    if (headerValue !== null) {
      newHeaders.set(headerName, headerValue);
    }
  });
}
```
