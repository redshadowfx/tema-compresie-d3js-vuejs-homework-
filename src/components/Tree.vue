<template>
  <div>
    <button type="click" @click="makeTree('huffman')">createHuffmanTree</button>
    <button type="click" @click="makeTree('shanon')">createShanonTree</button>
    <p>Code is:</p>
    <ul v-for="item in characterCodes" :key="item.character">
        <li>{{item.character}}:{{item.code}}</li>
    </ul>
  </div>
</template>

<script>
import * as d3 from "d3";
export default {
  props: ["message"],
  mounted() {
    this.resetFrequency();
  },
  data() {
    return {
      characterFrequency: {},
      treeData: {},
      characterCodes: [],
      validCharacters: "qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM., \"'"
    };
  },
  methods: {
    getCodes(){
        this.characterCodes = [];
        this.getCodesRecursive(this.treeData,"");
    },
    getCodesRecursive(currentTree,code){
        if(currentTree.children.length === 0){
            this.characterCodes.push({
                character: currentTree.name,
                code: code
            })
        }
        else{
            let leftChild = currentTree.children[0];
            let rightChild = currentTree.children[1];
            this.getCodesRecursive(leftChild,code+"0");
            this.getCodesRecursive(rightChild,code+"1");
        }
    },
    calculateTreeData(method, msg) {
      this.resetFrequency();
      [...msg].forEach(character => {
        this.characterFrequency[character] += 1;
      });
      // we need to select only the characters that accually appear int the message
      let workingFrequencies = [];
      for (let key of Object.keys(this.characterFrequency)) {
        if (this.characterFrequency[key] > 0)
          workingFrequencies.push({
            name: key,
            frequency: this.characterFrequency[key],
            children: []
          });
      }
      // sort the values descending
      workingFrequencies.sort((a, b) => {
        return b["frequency"] - a["frequency"];
      });

      //console.log(workingFrequencies);
      switch (method) {
        case "huffman":
          return this.huffmanTree(workingFrequencies);
        case "shanon":
          return this.shanonTree(workingFrequencies);
        default:
          console.log("some error happened with method selection!");
          return {};
      }
    },
    huffmanTree(tree) {
      while (tree.length > 1) {
        // we get the last two elements, and insert them in a node
        let lastIndex = tree.length - 1;
        let rightNode = tree[lastIndex];
        let leftNode = tree[lastIndex - 1];
        tree.pop();
        tree.pop();
        let currentNode = {
          // this version names them kinda ok,
          // but because the shanon version has no name i commented thi out 
          //name: leftNode["name"] + rightNode["name"],
          name:"",
          frequency: leftNode["frequency"] + rightNode["frequency"],
          children: [leftNode, rightNode]
        };

        // we need to insert the new node and have the array sorted
        // this could be optimised by inserting the new node in the right location without sorting
        tree.push(currentNode);
        tree.sort((a, b) => {
          return b["frequency"] - a["frequency"];
        });
      }
      return tree[0];
    },
    shanonTree(subTree) {
      if (subTree.length > 1) {
        // get the total frequency of this current subtree
        let totalFrequency = subTree.reduce(
          (accumulator, currentNode) => accumulator + currentNode["frequency"],
          0
        );
        let sum = 0;
        let i = 0;
        for (i; i < subTree.length && (sum <(totalFrequency / 2)); i += 1) {
          sum += subTree[i].frequency;
        }

        console.log(i);

        // we cut the array in half so we can recursively return to the left and right parts
        let leftSubTree = subTree.splice(0, i);
        let rightSubTree = subTree;

        return {
          name: "",
          frequency: totalFrequency,
          children: [
            this.shanonTree(leftSubTree),
            this.shanonTree(rightSubTree)
          ]
        };
      } else {
        return subTree[0];
      }
    },
    renderTree(treeData) {
      // Set the dimensions and margins of the diagram
      var margin = { top: 20, right: 90, bottom: 30, left: 90 },
        width = 3060 - margin.left - margin.right,
        height = 1000 - margin.top - margin.bottom;

      d3.select("svg").remove();

      // append the svg object to the body of the page
      // appends a 'group' element to 'svg'
      // moves the 'group' element to the top left margin
      var svg = d3
        .select("div")
        .append("svg")
        .attr("width", width + margin.right + margin.left)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

      var i = 0,
        duration = 750,
        root;

      // declares a tree layout and assigns the size
      var treemap = d3.tree().size([height, width]);

      // Assigns parent, children, height, depth
      root = d3.hierarchy(treeData, function(d) {
        return d.children;
      });
      root.x0 = height / 2;
      root.y0 = 0;

      // Collapse after the second level
      //root.children.forEach(collapse);

      update(root);

      // Collapse the node and all it's children
      function collapse(d) {
        if (d.children) {
          d._children = d.children;
          d._children.forEach(collapse);
          d.children = null;
        }
      }

      function update(source) {
        // Assigns the x and y position for the nodes
        var treeData = treemap(root);

        // Compute the new tree layout.
        var nodes = treeData.descendants(),
          links = treeData.descendants().slice(1);

        // Normalize for fixed-depth.
        nodes.forEach(function(d) {
          d.y = d.depth * 180;
        });

        // ****************** Nodes section ***************************

        // Update the nodes...
        var node = svg.selectAll("g.node").data(nodes, function(d) {
          return d.id || (d.id = ++i);
        });

        // Enter any new modes at the parent's previous position.
        var nodeEnter = node
          .enter()
          .append("g")
          .attr("class", "node")
          /* eslint no-unused-vars: 'off' */
          .attr("transform", function(d) {
            return "translate(" + source.y0 + "," + source.x0 + ")";
          })
          .on("click", click);

        // Add Circle for the nodes
        nodeEnter
          .append("circle")
          .attr("class", "node")
          .attr("r", 1e-6)
          .style("fill", function(d) {
            return d._children ? "lightsteelblue" : "#fff";
          });

        // Add labels for the nodes
        nodeEnter
          .append("text")
          .attr("dy", ".35em")
          .attr("x", function(d) {
            return d.children || d._children ? -13 : 13;
          })
          .attr("text-anchor", function(d) {
            return d.children || d._children ? "end" : "start";
          })
          .text(function(d) {
            return d.data.name;
          });

        // UPDATE
        var nodeUpdate = nodeEnter.merge(node);

        // Transition to the proper position for the node
        nodeUpdate
          .transition()
          .duration(duration)
          .attr("transform", function(d) {
            return "translate(" + d.y + "," + d.x + ")";
          });

        // Update the node attributes and style
        nodeUpdate
          .select("circle.node")
          .attr("r", 10)
          .style("fill", function(d) {
            return d._children ? "lightsteelblue" : "#fff";
          })
          .attr("cursor", "pointer");

        // Remove any exiting nodes
        var nodeExit = node
          .exit()
          .transition()
          .duration(duration)
          .attr("transform", function(d) {
            return "translate(" + source.y + "," + source.x + ")";
          })
          .remove();

        // On exit reduce the node circles size to 0
        nodeExit.select("circle").attr("r", 1e-6);

        // On exit reduce the opacity of text labels
        nodeExit.select("text").style("fill-opacity", 1e-6);

        // ****************** links section ***************************

        // Update the links...
        var link = svg.selectAll("path.link").data(links, function(d) {
          return d.id;
        });

        // Enter any new links at the parent's previous position.
        var linkEnter = link
          .enter()
          .insert("path", "g")
          .attr("class", "link")
          .attr("d", function(d) {
            var o = { x: source.x0, y: source.y0 };
            return diagonal(o, o);
          });

        // UPDATE
        var linkUpdate = linkEnter.merge(link);

        // Transition back to the parent element position
        linkUpdate
          .transition()
          .duration(duration)
          .attr("d", function(d) {
            return diagonal(d, d.parent);
          });

        // Remove any exiting links
        var linkExit = link
          .exit()
          .transition()
          .duration(duration)
          .attr("d", function(d) {
            var o = { x: source.x, y: source.y };
            return diagonal(o, o);
          })
          .remove();

        // Store the old positions for transition.
        nodes.forEach(function(d) {
          d.x0 = d.x;
          d.y0 = d.y;
        });

        // Creates a curved (diagonal) path from parent to the child nodes
        function diagonal(s, d) {
          let path = `M ${s.y} ${s.x}
            C ${(s.y + d.y) / 2} ${s.x},
              ${(s.y + d.y) / 2} ${d.x},
              ${d.y} ${d.x}`;

          return path;
        }

        // Toggle children on click.
        function click(d) {
          if (d.children) {
            d._children = d.children;
            d.children = null;
          } else {
            d.children = d._children;
            d._children = null;
          }
          update(d);
        }
      }
    },
    makeTree(method) {
      this.treeData = this.calculateTreeData(method, this.message);
      this.getCodes();
      this.renderTree(this.treeData);
    },
    resetFrequency() {
      // the valid characters
      [...this.validCharacters].forEach(
        val => {
          this.characterFrequency[val] = 0;
        }
      );
    }
  }
};
</script>

<style scoped>
ul, li{
    display:inline;
}
</style>