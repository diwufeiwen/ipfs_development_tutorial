
```
import React from 'react'
import ReactDOM from 'react-dom'
import Web3 from 'web3'
import './../css/index.css'

class App extends React.Component {
   constructor(props){
      super(props)
      this.state = {
         lastWinner: 0,
         timer: 0
      }
   }

voteNumber(number){
      console.log(number)
   }

render(){
      return (
         <div className="main-container">
            <h1>Bet for your best number and win huge amounts of Ether</h1>

<div className="block">
               <h4>Timer:</h4> &nbsp;
               <span ref="timer"> {this.state.timer}</span>
            </div>

<div className="block">
               <h4>Last winner:</h4> &nbsp;
               <span ref="last-winner">{this.state.lastWinner}</span>
            </div>

<hr/>

<h2>Vote for the next number</h2>
            <ul>
               <li onClick={() => {this.voteNumber(1)}}>1</li>
               <li onClick={() => {this.voteNumber(2)}}>2</li>
               <li onClick={() => {this.voteNumber(3)}}>3</li>
               <li onClick={() => {this.voteNumber(4)}}>4</li>
               <li onClick={() => {this.voteNumber(5)}}>5</li>
               <li onClick={() => {this.voteNumber(6)}}>6</li>
               <li onClick={() => {this.voteNumber(7)}}>7</li>
               <li onClick={() => {this.voteNumber(8)}}>8</li>
               <li onClick={() => {this.voteNumber(9)}}>9</li>
               <li onClick={() => {this.voteNumber(10)}}>10</li>
            </ul>
         </div>
      )
   }
}

ReactDOM.render(
   <App />,
   document.querySelector('#root')
)
```
