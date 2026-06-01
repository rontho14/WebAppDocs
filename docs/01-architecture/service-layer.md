# Service Layer (API)

All network access goes through a dedicated layer. Components never call `axios`
directly.

## Centralized client
```ts
// services/apiClient.ts
import axios from 'axios';
import { env } from '@/config/env';

export const apiClient = axios.create({
  baseURL: env.apiUrl,
  timeout: 10_000,
  headers: { 'Content-Type': 'application/json' },
});

apiClient.interceptors.request.use((cfg) => {
  const token = getAuthToken();
  if (token) cfg.headers.Authorization = `Bearer ${token}`;
  return cfg;
});

apiClient.interceptors.response.use(
  (r) => r,
  (err) => Promise.reject(normalizeError(err)),  // single error shape
);
```

## Services grouped by feature
```ts
// services/visitorService.ts
import { apiClient } from './apiClient';

export const visitorService = {
  get: (id: string) => apiClient.get<Visitor>(`/visitors/${id}`).then((r) => r.data),
  create: (data: VisitorInput) => apiClient.post<Visitor>('/visitors', data).then((r) => r.data),
};
```

## Consume via React Query (caching, retries, offline)
```ts
// features/checkin/hooks/useVisitor.ts
export const useVisitor = (id: string) =>
  useQuery({
    queryKey: ['visitor', id],
    queryFn: () => visitorService.get(id),
    staleTime: 60_000,
  });

export const useCreateVisitor = () => {
  const qc = useQueryClient();
  return useMutation({
    mutationFn: visitorService.create,
    onSuccess: () => qc.invalidateQueries({ queryKey: ['visitor'] }),
  });
};
```

## Rules
- One `apiClient`; base URL + headers configured once.
- One service object per domain (`visitorService`, `authService`).
- Services return typed domain data, not raw `AxiosResponse`.
- Normalize errors centrally (see [`../05-quality/error-handling-performance.md`](../05-quality/error-handling-performance.md)).
- Kiosks have flaky networks: set retries, timeouts, and offline cache via React Query.
