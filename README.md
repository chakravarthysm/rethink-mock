#rethink-mock
-----------------------
A mocking library for unit testing rethinkdb with sinon
spies and stubs.

## Install
```bash
npm install --save-dev rethink-mock
```

## Use

### Example
```javascript
// example using mocha, chai, sinon-chai and proxyquire
const rethinkmock = require('rethink-mock')
const proxyquire = require('proxyquire')
const chai = require('chai')
const expect = chai.expect
chai.use(require('sinon-chai'))

describe('userService', () => {
  let userService, r
  beforeEach(() => {
    r = rethinkmock.mock()
    userService = proxyquire('./userService', {
      'rethinkdb': r
    })
  })
  describe('#get', () => {
    it('connects', () => {
      return userService.get('some-id')
        .then(() => {
          expect(r.connect)
            .calledOnce
        })
    })
    it('calls the correct table', () => {
      return userService.get('some-id')
        .then(() => {
          expect(r.table)
            .calledOnce
            .calledWith('users')
        })
    })
    it('calls get with the passed in id', () => {
      return userService.get('some-id')
        .then(() => {
          expect(r.table.get)
            .calledOnce
            .calledWith('some-id')
        })
    })
    it('calls .run with the connection', () => {
      return userService.get('some-id')
        .then(() => {
          expect(r.mock.run)
            .calledOnce
            .calledWith(r.mock.connection)
        })
    })
    it('returns the user', () => {
      const alex = {firstName: 'Alex', lastName: 'DeLarge'}
      r.mock.run.returns(alex)

      return userService.get('some-id')
        .then(user => expect(user).to.eql(alex))
    })
  })
})
```

### Options

#### db: true
By default, ```.db``` is missing from the ```r``` object. This
is because the database is usually selected in the connection
and any ```.db``` found in the code is probably left over from
using the **Data Explorer**.

If you want to use ```.db```, call ```.mock({db: true})```
