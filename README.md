*This is in development and is not yet available.*

# AS2

Ruby implementation of the AS2 protocol used world wide for securely transferring Electronic documents.

## Business requirement

To be able to send out payloads using the AS2 format, you need the following items:

* A public certificate that is to be handed out to any business partners so that they can verify the payload is coming from you.
* A related private key *that you must keep safe and NOT show anybody* that is used to sign the payload.
* Your business partner's public certificate to encrypt the payload so that only they can decrypt it.
* An AS2 idenfitifer which is just a string that is used by your business partner to identify you.
* The HTTP based endpoint of your business partner(s) to know where to send the signed encrypted AS2 payload.

To be able to receive payloads using the AS2 format, you need the following:

* An HTTP endpoint that can receive the request and run this gem on receipt of an AS2 based HTTP request. Sinatra or Rails might be possibles for this.
* A public certificate that is to be handed out to any business partners so that they can encrypt the payload.
* A related private key so that you can decrypt the payload.

## Code requirements

* Ruby MRI 1.9.3 or 2.0.0

## Usage

This library will handle the whole AS2 process for sending out payloads, but leaves the logic of specifying what certificate and private key to use it as well as what the identifiers or you and your business partner are and where the endpoint is. This means that you can decide how to store this information and retrieve it per request.

The following is an example for sending out a payload over AS2:

```ruby
send = AS2.send_payload do
  private_key my_cert_instance # OpenSSL::X509::Certificate instance
  public_key xxx # from business partner for encryption
  require_mdn true
  endpoint "http://somehwete.com/as2"
  my_identifier "AS1"
  their_identifier "AS3"
end
# => AS2::Send::Request instance

response = send.call(payload) # this method raises normal HTTP exceptions and custom AS2 exceptions as well depending on whether it is a wire problem or an AS2 based problem
# => AS2::Send::Response instance
```

For receiving AS2 payloads you can use whatever HTTP server implementation you like but you will need to pass in the HTTP request body as a String, and the HTTP headers that relate to AS2 as a Hash.

```ruby
receive = AS2.receive_payload do
  certificate xxx
  private_key xxx
end
# => AS2::Recieve::Request instance

response = receive.call(headers, payload)
# => AS2::Receive::Response

response.body
# => String

response.mdn # you can acces the mdn that you may wish to send out.
# => AS2::MDN instance

# the mdn can be converted to an HTTP request for sending out.
response.mdn.to_request
```
