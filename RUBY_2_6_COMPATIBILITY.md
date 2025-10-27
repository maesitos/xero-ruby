# Ruby 2.6 Compatibility Changes

This document summarizes the changes made to ensure `xero-ruby` is compatible with Ruby 2.6 and its dependencies.

## Issues Fixed

### 1. Hash#compact Method Incompatibility (Ruby 2.3 compatibility)
**File:** `lib/xero-ruby/api_client.rb`
**Line:** ~68 (in `authorization_url` method)
**Problem:** `Hash#compact` was introduced in Ruby 2.4, but gemspec allows Ruby 2.3+
**Solution:** Replaced `.compact` with `.reject { |k, v| v.nil? }`

```ruby
# Before:
}.compact

# After:
params = params.reject { |k, v| v.nil? }
```

### 2. Faraday Authorization Middleware Incompatibility
**File:** `lib/xero-ruby/api_client.rb`
**Line:** ~308 (in `call_api` method)
**Problem:** `conn.request(:authorization, :basic, ...)` causes ArgumentError with wrong number of arguments in Faraday 1.x
**Solution:** Use `conn.basic_auth()` with fallback to manual Authorization header

```ruby
# Before:
conn.request(:authorization, :basic, config.username, config.password)

# After:
begin
  conn.basic_auth(@config.username, @config.password) if @config.username || @config.password
rescue NoMethodError
  # basic_auth not available; build_request will set Authorization header.
end
```

### 3. Basic Auth Fallback in Request Headers
**File:** `lib/xero-ruby/api_client.rb`
**Line:** ~380 (in `build_request` method)
**Problem:** If Faraday connection doesn't have basic auth configured, requests fail
**Solution:** Added fallback to set Authorization header manually in build_request

```ruby
# Added fallback:
if (@config.username || @config.password) && !header_params['Authorization']
  require 'base64'
  credentials = Base64.strict_encode64("#{@config.username}:#{@config.password}")
  header_params['Authorization'] = "Basic #{credentials}"
end
```

## Dependencies Verification

### Runtime Dependencies (all compatible with Ruby 2.6):
- `faraday ~> 1.0, >= 1.0.1` ✅ Compatible with Ruby 2.6
- `json >= 1.8.6, < 2.0` ✅ Compatible with Ruby 2.6  
- `json-jwt >= 1.5.0, < 1.13.0` ✅ Compatible with Ruby 2.6

### Development Dependencies:
- `rspec ~> 3.6, >= 3.6.0` ✅ Compatible with Ruby 2.6

## Testing

To test the fixes:

```bash
# Test the authorization_url method (should work without errors)
xero_client = XeroRuby::ApiClient.new(credentials: your_credentials)
puts xero_client.authorization_url

# Test API calls (should work without ArgumentError)
xero_client.set_token_set(your_token_set)
connections = xero_client.connections
last_conn = xero_client.last_connection
```

## Compatibility Notes

- **Ruby versions supported:** 2.3+ (as per gemspec, but now truly compatible)
- **Faraday versions supported:** 1.x (as specified in gemspec)
- **Backward compatibility:** All changes maintain backward compatibility with existing code
- **Error handling:** Added graceful fallbacks for different Faraday versions

The changes ensure that the gem works correctly across different Ruby versions and Faraday configurations without breaking existing functionality.