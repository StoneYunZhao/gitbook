# Code Snippet

## Do anything timeout

```go
func DoTimeout(ctx context.Context, timeout time.Duration, f func() (interface{}, error)) (interface{}, error) {
	ctx, cancel := context.WithTimeout(ctx, timeout)
	defer cancel()

	var resp interface{}
	var err error
	var lock sync.Mutex

	done := make(chan struct{})
	panicChain := make(chan interface{}, 1)

	go func() {
		defer func() {
			if p := recover(); p != nil {
				panicChain <- p
			}
		}()

		r, e := f()

		lock.Lock()
		defer lock.Unlock()
		resp = r
		err = e

		close(done)
	}()

	select {
	case <-done:
		lock.Lock()
		defer lock.Unlock()
		return resp, err
	case p := <-panicChain:
		return nil, fmt.Errorf("call function panic: %v", p)
	case <-ctx.Done():
		return nil, ctx.Err()
	}
}
```
